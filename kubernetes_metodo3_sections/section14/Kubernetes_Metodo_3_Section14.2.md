## Listar los pods del espacios de nombres alpha
kubectl get pods -o wide -n alpha 
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
mysql                           1/1     Running   0          2m30s   10.22.0.9    controlplane   [none]           [none]
webapp-mysql-6ddf84655d-rwplb   1/1     Running   0          2m30s   10.22.0.10   controlplane   [none]           [none]

## Obtener los deploy de todos los espacios de nombres
kubectl get deployments.apps -A
NAMESPACE     NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
alpha         webapp-mysql             1/1     1            1           6m53s
kube-system   coredns                  1/1     1            1           13m
kube-system   local-path-provisioner   1/1     1            1           13m
kube-system   metrics-server           1/1     1            1           13m
kube-system   traefik                  1/1     1            1           13m

## Extraer el yaml del servicio defectuoso-1 y editarlo para corregir el fallo
kubectl get service -n alpha mysql -o yaml > mysql_service_file_1.yaml
nano mysql_service_file_1.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"mysql-service","namespace":"alpha"},"spec":{"ports":[{"port":3306,"targetPort":3306}],"selector":{"name":"mysql"}}}
  creationTimestamp: "2026-08-11T10:58:40Z"
  name: mysql-service
  namespace: alpha
  resourceVersion: "845"
  uid: ae3c63ae-29f4-427b-abaf-d7e297abd908
spec:
  clusterIP: 10.43.151.82
  clusterIPs:
  - 10.43.151.82
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    name: mysql
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

## Borra el servicio defectuoso-1
kubectl delete service -n alpha mysql 
service "mysql" deleted from alpha namespace

## Relanza el servicio corregido-1
kubectl apply -f mysql_service_file.yaml 
service/mysql-service created

## --> El servicio está conectado con éxito

------------------------------------------------------------

## Consultar los pods del espacio de nombres beta
kubectl get pods -o wide -n beta 
NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
mysql                           1/1     Running   0          53s   10.22.0.11   controlplane   [none]           [none]
webapp-mysql-6ddf84655d-sxhpf   1/1     Running   0          53s   10.22.0.12   controlplane   [none]           [none]

## Consutlar el servicio del espacio de nombres beta (Servicio web)
kubectl get service -n beta web-service -o wide 
NAME          TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE     SELECTOR
web-service   NodePort   10.43.146.36   [none]        8080:30081/TCP   3m17s   name=webapp-mysql

## Consutlar el servicio del espacio de nombres beta (Servicio de base de datos)
kubectl get service -n beta mysql-service -o wide 
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE     SELECTOR
mysql-service   ClusterIP   10.43.32.49   [none]        3306/TCP   4m47s   name=mysql

## Describir el servicio mysql-service alojado en el ns beta
kubectl describe service -n beta mysql-service
Name:                     mysql-service
Namespace:                beta
Labels:                   [none]
Annotations:              [none]
Selector:                 name=mysql
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.32.49
IPs:                      10.43.32.49
Port:                     <unset>  3306/TCP
TargetPort:               8080/TCP
Endpoints:                10.22.0.11:8080
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   [none]

## Extrae el yaml del mysql-service defectuoso-2 para corregirlo
kubectl get services -n beta mysql-service -o yaml > mysql_service_file_2.yaml 
nano mysql_service_file_2.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"mysql-service","namespace":"beta"},"spec":{"ports":[{"port":3306,"targetPort":8080}],"selector":{"name":"mysql"}}}
  creationTimestamp: "2026-08-11T11:17:12Z"
  name: mysql-service
  namespace: beta
  resourceVersion: "1257"
  uid: 2bda3fa7-cdb8-47ff-951c-3ac9d19d9b89
spec:
  clusterIP: 10.43.32.49
  clusterIPs:
  - 10.43.32.49
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    name: mysql
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

## Borra el servicio defectuoso-2
kubectl delete services -n beta mysql-service
service "mysql-service" deleted from beta namespace

## Relanza el servicio-2 corregido
kubectl apply -f mysql_service_file.yaml 
service/mysql-service created

## --> El servicio está conectado con éxito

----------------------------------------------------------------------------------

## Consulta los pods del espacio de nombres gamma
kubectl get pods -o wide -n gamma 
NAME                            READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
mysql                           1/1     Running   0          52s   10.22.0.13   controlplane   [none]           [none]
webapp-mysql-6ddf84655d-pnqhf   1/1     Running   0          51s   10.22.0.14   controlplane   [none]           [none]

## Extrae el yaml del mysql-service defectuoso-3 del ns gamma y corregirlo 
kubectl get service -n gamma mysql-service -o yaml > mysql_service_file_3.yaml 
nano mysql_service_file_3.yaml 
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"mysql-service","namespace":"gamma"},"spec":{"ports":[{"port":3306,"targetPort":3306}],"selector":{"name":"sql00001"}}}
  creationTimestamp: "2026-08-11T11:29:47Z"
  name: mysql-service
  namespace: gamma
  resourceVersion: "1582"
  uid: e557dfd8-5cd7-46fd-ace6-0f8e72fecd83
spec:
  clusterIP: 10.43.28.28
  clusterIPs:
  - 10.43.28.28
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    name: mysql
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

## Borra el servcicio defectuoso-3
kubectl delete service -n gamma mysql-service
service "mysql-service" deleted from gamma namespace

## Relanza el servicio corregido
kubectl apply -f mysql_service_file.yaml 
service/mysql-service created

## --> El servicio está conectado con éxito

-----------------------------------------------------------------------------

## Consulta los pods del ns delta
kubectl get pods -n delta -o wide 
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
mysql                           1/1     Running   0          2m59s   10.22.0.15   controlplane   [none]           [none]
webapp-mysql-69867bff7d-ntd5m   1/1     Running   0          2m59s   10.22.0.16   controlplane   [none]           [none]

## Consultar los servicios del ns delta
kubectl get service -n delta -o wide 
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE   SELECTOR
mysql-service   ClusterIP   10.43.33.100    [none]        3306/TCP         62s   name=mysql
web-service     NodePort    10.43.239.178   [none]        8080:30081/TCP   62s   name=webapp-mysql

## Consultar el deploy del ns delta
kubectl get deploy -n delta -o wide 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS     IMAGES                         SELECTOR
webapp-mysql   1/1     1            1           2m16s   webapp-mysql   mmumshad/simple-webapp-mysql   name=webapp-mysql 

## Describir el deploy del ns delta
kubectl describe deploy -n delta | grep -A5 Environment
    Environment:
      DB_Host:      mysql-service
      DB_User:      sql-user # --> change the bad_name for [root]
      DB_Password:  paswrd
    Mounts:         [none]
  Volumes:          [none]

## Extrae el yaml del pod defectuoso-4 y editarlo
kubectl get deploy -n delta webapp-mysql -o yaml > webapp-mysql_deploy_file_4.yaml 

nano webapp-mysql_deploy_file_4.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"name":"webapp-mysql"},"name":"webapp-mysql","namespace":"delta"},"spec":{"selector":{"matchLabels":{"name":"webapp-mysql"}},"template":{"metadata":{"labels":{"name":"webapp-mysql"},"name":"webapp-mysql"},"spec":{"containers":[{"env":[{"name":"DB_Host","value":"mysql-service"},{"name":"DB_User","value":"sql-user"},{"name":"DB_Password","value":"paswrd"}],"image":"mmumshad/simple-webapp-mysql","name":"webapp-mysql","ports":[{"containerPort":8080}]}]}}}}
  creationTimestamp: "2026-08-12T09:32:19Z"
  generation: 1
  labels:
    name: webapp-mysql
  name: webapp-mysql
  namespace: delta
  resourceVersion: "1160"
  uid: fd398fda-d4c5-4453-af4d-0b80f243ea74
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: webapp-mysql
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: webapp-mysql
      name: webapp-mysql
    spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql-service
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
        image: mmumshad/simple-webapp-mysql
        imagePullPolicy: Always
        name: webapp-mysql
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2026-08-12T09:32:21Z"
    lastUpdateTime: "2026-08-12T09:32:21Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-08-12T09:32:19Z"
    lastUpdateTime: "2026-08-12T09:32:21Z"
    message: ReplicaSet "webapp-mysql-69867bff7d" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  terminatingReplicas: 0
  updatedReplicas: 1

## Aplica los cambios
kubectl apply -f webapp-mysql_deploy_file_4.yaml 
deployment.apps/webapp-mysql configured

## --> El servicio está conectado con éxito

-------------------------------------------------------------------------------

## Consulta del deploy con el entorno de mysql defectuoso
kubectl get deployments.apps -n epsilon -o wide 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS     IMAGES                         SELECTOR
webapp-mysql   1/1     1            1           7m55s   webapp-mysql   mmumshad/simple-webapp-mysql   name=webapp-mysql

## Describe el deploy webapp-mysql y localiza algún defecto
kubectl describe deployments.apps -n epsilon | grep -A5 Environment
    Environment:
      DB_Host:      mysql-service
      DB_User:      sql-user # Change this value for [root]
      DB_Password:  paswrd
    Mounts:         [none]
  Volumes:          [none]

## Extrae el yaml del deploy del epsilon defectuoso-5.1 y corrige el fallo 
kubectl get deployments.apps -n epsilon -o yaml > webapp-mysql_5.1.yaml

## Edita el fichero
nano webapp-mysql_5.1.yaml
apiVersion: v1
items:
- apiVersion: apps/v1
  kind: Deployment
  metadata:
    annotations:
      deployment.kubernetes.io/revision: "1"
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"name":"webapp-mysql"},"name":"webapp-mysql","namespace":"epsilon"},"spec":{"selector":{"matchLabels":{"name":"webapp-mysql"}},"template":{"metadata":{"labels":{"name":"webapp-mysql"},"name":"webapp-mysql"},"spec":{"containers":[{"env":[{"name":"DB_Host","value":"mysql-service"},{"name":"DB_User","value":"sql-user"},{"name":"DB_Password","value":"paswrd"}],"image":"mmumshad/simple-webapp-mysql","name":"webapp-mysql","ports":[{"containerPort":8080}]}]}}}}
    creationTimestamp: "2026-08-12T11:10:57Z"
    generation: 1
    labels:
      name: webapp-mysql
    name: webapp-mysql
    namespace: epsilon
    resourceVersion: "1267"
    uid: f93b7f4e-d73a-4cb6-a80b-fcac719ac8c8
  spec:
    progressDeadlineSeconds: 600
    replicas: 1
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        name: webapp-mysql
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        labels:
          name: webapp-mysql
        name: webapp-mysql
      spec:
        containers:
        - env:
          - name: DB_Host
            value: mysql-service
          - name: DB_User
            value: root
          - name: DB_Password
            value: paswrd
          image: mmumshad/simple-webapp-mysql
          imagePullPolicy: Always
          name: webapp-mysql
          ports:
          - containerPort: 8080
            protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
  status:
    availableReplicas: 1
    conditions:
    - lastTransitionTime: "2026-08-12T11:10:59Z"
      lastUpdateTime: "2026-08-12T11:10:59Z"
      message: Deployment has minimum availability.
      reason: MinimumReplicasAvailable
      status: "True"
      type: Available
    - lastTransitionTime: "2026-08-12T11:10:57Z"
      lastUpdateTime: "2026-08-12T11:10:59Z"
      message: ReplicaSet "webapp-mysql-69867bff7d" has successfully progressed.
      reason: NewReplicaSetAvailable
      status: "True"
      type: Progressing
    observedGeneration: 1
    readyReplicas: 1
    replicas: 1
    terminatingReplicas: 0
    updatedReplicas: 1
kind: List
metadata:
  resourceVersion: ""

## Aplica el cambio
kubectl apply -f webapp-mysql_5.1.yaml
deployment.apps/webapp-mysql configured

## Describe el pod mysql del ns epsilon y filtrar por MYSQL_ROOT_PASSWORD. El servicio necesita reparar un parámetro en el pod
kubectl -n epsilon describe pod mysql  | grep MYSQL_ROOT_PASSWORD 
      MYSQL_ROOT_PASSWORD:  passwooooorrddd

## Extraer el yaml del pod mysql
kubectl get pods -n epsilon mysql -o yaml > mysql_pod_5.2.yaml

## Edita el fichero
nano mysql_pod_5.2.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"name":"mysql"},"name":"mysql","namespace":"epsilon"},"spec":{"containers":[{"env":[{"name":"MYSQL_ROOT_PASSWORD","value":"passwooooorrddd"}],"image":"mysql:5.6","name":"mysql","ports":[{"containerPort":3306}]}]}}
  creationTimestamp: "2026-08-12T09:41:48Z"
  generation: 1
  labels:
    name: mysql
  name: mysql
  namespace: epsilon
  resourceVersion: "1456"
  uid: ece1dbf4-a6bc-48dc-89ba-61d733b157a7
spec:
  containers:
  - env:
    - name: MYSQL_ROOT_PASSWORD
      value: paswrd
    image: mysql:5.6
    imagePullPolicy: IfNotPresent
    name: mysql
    ports:
    - containerPort: 3306
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-28p5g
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-28p5g
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T09:41:49Z"
    observedGeneration: 1
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T09:41:48Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T09:41:49Z"
    observedGeneration: 1
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T09:41:49Z"
    observedGeneration: 1
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T09:41:48Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://8f3b66dd1cda84db24aef60b09dada31307fefd734019d9b1fe8f722a17484fa
    image: docker.io/library/mysql:5.6
    imageID: docker.io/library/mysql@sha256:20575ecebe6216036d25dab5903808211f1e9ba63dc7825ac20cb975e34cfcae
    lastState: {}
    name: mysql
    ready: true
    resources: {}
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-08-12T09:41:49Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-28p5g
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 10.244.147.148
  hostIPs:
  - ip: 10.244.147.148
  observedGeneration: 1
  phase: Running
  podIP: 10.22.0.18
  podIPs:
  - ip: 10.22.0.18
  qosClass: BestEffort
  startTime: "2026-08-12T09:41:48Z"

## Borra el pod para evitar el bloqueo de seguirad
kubectl delete pods -n epsilon mysql 
pod "mysql" deleted from epsilon namespace

## Aplica los cambios
kubectl apply -f mysql_pod_5.2.yaml 
pod/mysql created

## --> Este servicio está conectado con éxito

-------------------------------------------------------

## Consultar los servicios activos en el ns zeta
kubectl get service -n zeta -o wide 
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE     SELECTOR
mysql-service   ClusterIP   10.43.219.80   [none]        3306/TCP         2m24s   name=mysql
web-service     NodePort    10.43.12.108   [none]        8080:30088/TCP   2m24s   name=webapp-mysql # --> This port is wrong. The por to listen is [30081]

## Consultar los pods en el ns zeta
kubectl get pods -n zeta -o wide 
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
mysql                           1/1     Running   0          3m46s   10.22.0.22   controlplane   [none]           [none]
webapp-mysql-69867bff7d-trwz7   1/1     Running   0          3m45s   10.22.0.23   controlplane   [none]           [none]

## Consultar el deploy del espacio de nombres zeta
kubectl get deployments.apps -n zeta -o wide 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE    CONTAINERS     IMAGES                         SELECTOR
webapp-mysql   1/1     1            1           4m9s   webapp-mysql   mmumshad/simple-webapp-mysql   name=webapp-mysql

## Consultar el registro del pod webapp
kubectl logs -n zeta pods/webapp-mysql-69867bff7d-trwz7 
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)

## Consultar el registro del pod mysql
kubectl logs -n zeta pods/mysql | grep Warning
2026-08-12 10:12:28 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2026-08-12 10:12:29 94 [Warning] InnoDB: New log files created, LSN=45781
2026-08-12 10:12:29 94 [Warning] InnoDB: Creating foreign key constraint system tables.
2026-08-12 10:12:30 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2026-08-12 10:12:33 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2026-08-12 10:12:33 142 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 558b3e39-9636-11f1-a107-da6b8f237342.

## Filtra la descripción del deploy para localizar una linea defectuosa
kubectl describe deploy -n zeta webapp-mysql | grep -A5 Env
    Environment:
      DB_Host:      mysql-service
      DB_User:      sql-user # --> Change this value for [root]
      DB_Password:  paswrd
    Mounts:         [none]
  Volumes:          [none]

## Extrae el yaml del deploy para corregirlo
kubectl get deploy -n zeta webapp-mysql -o yaml > webapp-mysql_6.1.yaml

## Edita el fichero para la corrección
nano webapp-mysql_6.1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"name":"webapp-mysql"},"name":"webapp-mysql","namespace":"zeta"},"spec":{"selector":{"matchLabels":{"name":"webapp-mysql"}},"template":{"metadata":{"labels":{"name":"webapp-mysql"},"name":"webapp-mysql"},"spec":{"containers":[{"env":[{"name":"DB_Host","value":"mysql-service"},{"name":"DB_User","value":"sql-user"},{"name":"DB_Password","value":"paswrd"}],"image":"mmumshad/simple-webapp-mysql","name":"webapp-mysql","ports":[{"containerPort":8080}]}]}}}}
  creationTimestamp: "2026-08-12T11:15:17Z"
  generation: 1
  labels:
    name: webapp-mysql
  name: webapp-mysql
  namespace: zeta
  resourceVersion: "1490"
  uid: adec9236-e505-4109-8eab-7fdf8e2b9de8
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: webapp-mysql
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: webapp-mysql
      name: webapp-mysql
    spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql-service
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
        image: mmumshad/simple-webapp-mysql
        imagePullPolicy: Always
        name: webapp-mysql
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2026-08-12T11:15:18Z"
    lastUpdateTime: "2026-08-12T11:15:18Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-08-12T11:15:17Z"
    lastUpdateTime: "2026-08-12T11:15:18Z"
    message: ReplicaSet "webapp-mysql-69867bff7d" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  terminatingReplicas: 0
  updatedReplicas: 1

## Aplica el cambio
kubectl apply -f webapp-mysql_6.1.yaml 
deployment.apps/webapp-mysql configured

## Extrae el yaml del servicio web-service en el espacio de nombres zeta
kubectl get service -n zeta -o yaml web-service > web-service_6.2.yaml

## Edita el fichero
nano web-service_6.2.yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"web-service","namespace":"zeta"},"spec":{"ports":[{"nodePort":30088,"port":8080,"targetPort":8080}],"selector":{"name":"webapp-mysql"},"type":"NodePort"}}
  creationTimestamp: "2026-08-12T11:15:17Z"
  name: web-service
  namespace: zeta
  resourceVersion: "1473"
  uid: a54c00c2-7033-4d72-8e92-65b96575defc
spec:
  clusterIP: 10.43.180.144
  clusterIPs:
  - 10.43.180.144
  externalTrafficPolicy: Cluster
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - nodePort: 30081
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    name: webapp-mysql
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}

## Borra el servicio defectuoso para re-aplicarlo
kubectl delete service -n zeta web-service 
service "web-service" deleted from zeta namespace

## Aplica los cambios
kubectl apply -f web-service_6.2.yaml 
service/web-service created

## El pod myslq del ns zeta tiene un parámetro defectuoso
kubectl describe pod -n zeta mysql | grep PASS
      MYSQL_ROOT_PASSWORD:  passwooooorrddd

## Extrae el yaml del pod mysql del ns zeta
kubectl get pod -n zeta mysql -o yaml > mysql_pod_6.3.yaml

## Edita el fichero
nano mysql_pod_6.3.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"name":"mysql"},"name":"mysql","namespace":"zeta"},"spec":{"containers":[{"env":[{"name":"MYSQL_ROOT_PASSWORD","value":"passwooooorrddd"}],"image":"mysql:5.6","name":"mysql","ports":[{"containerPort":3306}]}]}}
  creationTimestamp: "2026-08-12T11:15:17Z"
  generation: 1
  labels:
    name: mysql
  name: mysql
  namespace: zeta
  resourceVersion: "1483"
  uid: f9c3c58a-35b0-4b45-956d-b729c83bad52
spec:
  containers:
  - env:
    - name: MYSQL_ROOT_PASSWORD
      value: paswrd
    image: mysql:5.6
    imagePullPolicy: IfNotPresent
    name: mysql
    ports:
    - containerPort: 3306
      protocol: TCP
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-n96hm
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: kube-api-access-n96hm
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T11:15:18Z"
    observedGeneration: 1
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T11:15:17Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T11:15:18Z"
    observedGeneration: 1
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T11:15:18Z"
    observedGeneration: 1
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T11:15:17Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://4c301cd1c78728890d8e73f1a2f2c7662d8d79d40a8ad5338612eb6080320d44
    image: docker.io/library/mysql:5.6
    imageID: docker.io/library/mysql@sha256:20575ecebe6216036d25dab5903808211f1e9ba63dc7825ac20cb975e34cfcae
    lastState: {}
    name: mysql
    ready: true
    resources: {}
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-08-12T11:15:18Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-n96hm
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 10.244.154.24
  hostIPs:
  - ip: 10.244.154.24
  observedGeneration: 1
  phase: Running
  podIP: 10.22.0.23
  podIPs:
  - ip: 10.22.0.23
  qosClass: BestEffort
  startTime: "2026-08-12T11:15:17Z"

## Borra el pod para aplicarlo
kubectl delete pod -n zeta mysql 
pod "mysql" deleted from zeta namespace

## Aplica los cambios
kubectl apply -f mysql_pod_6.3.yaml 
pod/mysql created

## --> Este servicio está conectado con éxito
