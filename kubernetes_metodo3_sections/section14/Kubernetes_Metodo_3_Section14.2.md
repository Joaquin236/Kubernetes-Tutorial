## Listar los pods del espacios de nombres alpha
kubectl get pods -o wide -n alpha 
NAME                            READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
mysql                           1/1     Running   0          2m30s   10.22.0.9    controlplane   <none>           <none>
webapp-mysql-6ddf84655d-rwplb   1/1     Running   0          2m30s   10.22.0.10   controlplane   <none>           <none>

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
mysql                           1/1     Running   0          53s   10.22.0.11   controlplane   <none>           <none>
webapp-mysql-6ddf84655d-sxhpf   1/1     Running   0          53s   10.22.0.12   controlplane   <none>           <none>

## Consutlar el servicio del espacio de nombres beta (Servicio web)
kubectl get service -n beta web-service -o wide 
NAME          TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE     SELECTOR
web-service   NodePort   10.43.146.36   <none>        8080:30081/TCP   3m17s   name=webapp-mysql

## Consutlar el servicio del espacio de nombres beta (Servicio de base de datos)
kubectl get service -n beta mysql-service -o wide 
NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE     SELECTOR
mysql-service   ClusterIP   10.43.32.49   <none>        3306/TCP   4m47s   name=mysql

## Describir el servicio mysql-service alojado en el ns beta
kubectl describe service -n beta mysql-service
Name:                     mysql-service
Namespace:                beta
Labels:                   <none>
Annotations:              <none>
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
Events:                   <none>

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
mysql                           1/1     Running   0          52s   10.22.0.13   controlplane   <none>           <none>
webapp-mysql-6ddf84655d-pnqhf   1/1     Running   0          51s   10.22.0.14   controlplane   <none>           <none>

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
mysql                           1/1     Running   0          2m59s   10.22.0.15   controlplane   <none>           <none>
webapp-mysql-69867bff7d-ntd5m   1/1     Running   0          2m59s   10.22.0.16   controlplane   <none>           <none>

## Extrae el yaml del pod defectuoso-4
kubectl get pods -n delta mysql -o yaml > mysql_service_file_4.yaml 
nano mysql_service_file_4.yaml 

## --> Este cluster está fuera de servicio 

-------------------------------------------------------------------------------

## Describe el pod mysql del ns epsilon y filtrar por MYSQL_ROOT_PASSWORD
kubectl -n epsilon describe pod mysql  | grep MYSQL_ROOT_PASSWORD 
      MYSQL_ROOT_PASSWORD:  passwooooorrddd

## Extrae el yaml del pod mysql defectuoso-5.1 y corrige el fallo 
kubectl get pods -n epsilon mysql -o yaml > mysql_service_file_5.1.yaml
nano mysql_service_file.yaml 
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"name":"mysql"},"name":"mysql","namespace":"epsilon"},"spec":{"containers":[{"env":[{"name":"MYSQL_ROOT_PASSWORD","value":"passwooooorrddd"}],"image":"mysql:5.6","name":"mysql","ports":[{"containerPort":3306}]}]}}
  creationTimestamp: "2026-08-11T11:48:12Z"
  generation: 1
  labels:
    name: mysql
  name: mysql
  namespace: epsilon
  resourceVersion: "2140"
  uid: 7c5d994b-b322-41f4-be52-349c64ab57f6
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
      name: kube-api-access-cfjnj
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
  - name: kube-api-access-cfjnj
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
    lastTransitionTime: "2026-08-11T11:48:13Z"
    observedGeneration: 1
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:12Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:13Z"
    observedGeneration: 1
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:13Z"
    observedGeneration: 1
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:12Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://971857c1fe2ad518fe1603c3b805499f517c09338cf6d0f8ee30aca11a3168a5
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
        startedAt: "2026-08-11T11:48:13Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-cfjnj
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 10.244.220.199
  hostIPs:
  - ip: 10.244.220.199
  observedGeneration: 1
  phase: Running
  podIP: 10.22.0.18
  podIPs:
  - ip: 10.22.0.18
  qosClass: BestEffort
  startTime: "2026-08-11T11:48:12Z"

## Borra el pod de mysql
kubectl delete pods -n epsilon mysql 
pod "mysql" deleted from epsilon namespace


## Extrae el yaml del pod webapp defectuoso-5.2 y corrige el fallo
kubectl get pods -n epsilon webapp-mysql-69867bff7d-km5f9 -o yaml > webapp_file_5.2.yaml
nano webapp_file_5.2.yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2026-08-11T11:48:12Z"
  generateName: webapp-mysql-69867bff7d-
  generation: 1
  labels:
    name: webapp-mysql
    pod-template-hash: 69867bff7d
  name: webapp-mysql-69867bff7d-km5f9
  namespace: epsilon
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: webapp-mysql-69867bff7d
    uid: d9a368ef-6e77-454f-982a-135ba4223235
  resourceVersion: "2147"
  uid: b605d60c-a01a-4103-8be0-a9386f4545bc
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
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-9cpxf
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
  - name: kube-api-access-9cpxf
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
    lastTransitionTime: "2026-08-11T11:48:14Z"
    observedGeneration: 1
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:12Z"
    observedGeneration: 1
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:14Z"
    observedGeneration: 1
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:14Z"
    observedGeneration: 1
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-08-11T11:48:12Z"
    observedGeneration: 1
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: containerd://d592172b64182a49a20f115fd2d7935bdf52543657437463fae7545f97dc520d
    image: docker.io/mmumshad/simple-webapp-mysql:latest
    imageID: docker.io/mmumshad/simple-webapp-mysql@sha256:d4d0c03fcb76cee6ae2511fa7f3f6b7090f0c5e0cb3f276687f9ddf2c689cc09
    lastState: {}
    name: webapp-mysql
    ready: true
    resources: {}
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2026-08-11T11:48:14Z"
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-9cpxf
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 10.244.220.199
  hostIPs:
  - ip: 10.244.220.199
  observedGeneration: 1
  phase: Running
  podIP: 10.22.0.19
  podIPs:
  - ip: 10.22.0.19
  qosClass: BestEffort
  startTime: "2026-08-11T11:48:12Z"

## Aplica los pods del mysql y webapp
kubectl apply -f my_service_file.yaml ; kubectl apply -f webapp_file.yaml 
pod/mysql created
Warning: resource pods/webapp-mysql-69867bff7d-km5f9 is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
The Pod "webapp-mysql-69867bff7d-km5f9" is invalid: spec: Forbidden: pod updates may not change fields other than `spec.containers[*].image`,`spec.initContainers[*].image`,`spec.activeDeadlineSeconds`,`spec.tolerations` (only additions to existing tolerations),`spec.terminationGracePeriodSeconds` (allow it to be set to 1 if it was previously negative)
@@ -114,7 +114,7 @@
     },
     {
      "Name": "DB_User",
-     "Value": "sql-user",
+     "Value": "root",
      "ValueFrom": null
     },
     {

## Task Skipped
