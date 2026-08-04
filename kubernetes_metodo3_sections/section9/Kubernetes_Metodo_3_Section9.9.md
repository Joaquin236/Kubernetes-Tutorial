## Consultar los pods, enfocarse en kube-system y filtrar por core
kubectl get pods -o wide -n kube-system | grep core
coredns-6f6c7df987-4rpj2               1/1     Running   0          12m   172.17.0.3      controlplane   [none]           [none]
coredns-6f6c7df987-sjlhj               1/1     Running   0          12m   172.17.0.2      controlplane   [none]           [none]
## There're two PODS at/on/in the cluster

## Consultar los servicios, enfocarse en kube-system
kubectl get service -o wide -n kube-system 
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
kube-dns   ClusterIP   172.20.0.10   [none]        53/UDP,53/TCP,9153/TCP   14m   k8s-app=kube-dns
##  IP is --> 172.20.0.10

## Consultar el deploy de kube-system
kubectl get deployments.apps -n kube-system -o wide 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
coredns   2/2     2            2           17m   coredns      registry.k8s.io/coredns/coredns:v1.10.1   k8s-app=kube-dns

## Localizar el path del deploy coredns
kubectl describe deployments.apps coredns -n kube-system | grep -A2 Args
    Args:
      -conf
      /etc/coredns/Corefile


## Buscar la ruta del fichero Core en la descripcion del deploy en el ns kube-system
kubectl describe deployments.apps -n kube-system | grep Core
      /etc/coredns/Corefile

## Consultar los configmaps del ns kube-system
kubectl get configmaps -n kube-system 
NAME                                                   DATA   AGE
coredns                                                1      20m
extension-apiserver-authentication                     6      20m
kube-apiserver-legacy-service-account-token-tracking   1      20m
kube-proxy                                             2      20m
kube-root-ca.crt                                       1      20m
kubeadm-config                                         1      20m
kubelet-config                                         1      20m

## Localizar la raiz de dominio en la descripcion del coredns en configmaps en el ns kube-system
kubectl describe configmaps coredns -n kube-system | grep cluster
    kubernetes cluster.local in-addr.arpa ip6.arpa

## Consultar los servicios en default y en payroll
kubectl get pods -n default -o wide ; kubectl get pods -n payroll -o wide 
NAME                READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
hr                  1/1     Running   0          10m   172.17.0.6   controlplane   [none]           [none]
simple-webapp-1     1/1     Running   0          10m   172.17.0.7   controlplane   [none]           [none]
simple-webapp-122   1/1     Running   0          10m   172.17.0.8   controlplane   [none]           [none]
test                1/1     Running   0          10m   172.17.0.5   controlplane   [none]           [none]
NAME   READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
web    1/1     Running   0          10m   172.17.0.4   controlplane   [none]           [none]

## Consultar los servicios
kubectl get service -o wide 
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
kubernetes     ClusterIP   172.20.0.1       [none]        443/TCP        25m   [none]
test-service   NodePort    172.20.244.221   [none]        80:30080/TCP   13m   name=test
web-service    ClusterIP   172.20.100.237   [none]        80/TCP         13m   name=hr

kubectl get service -o wide -n payroll 
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
web-service   ClusterIP   172.20.111.105   [none]        80/TCP    15m   name=web


## Acceder a las webs de los servicios
curl http://172.20.47.241
 This is the HR server!
----------------------------
curl http://172.20.161.141 #--> The web_file is very big

## Localizar los pods de todos los ns y filtrar por pay
kubectl get pods --all-namespaces -o wide | grep pay
payroll        mysql                                  1/1     Running   0          3m22s   172.17.0.10     controlplane   [none]           [none]
payroll        web                                    1/1     Running   0          21m     172.17.0.4      controlplane   [none]           [none]

## Localizar el ns pay
kubectl get namespaces | grep pay
payroll           Active   25m

## Consultar los pods del ns payroll
kubectl get pods -n payroll -o wide 
NAME    READY   STATUS    RESTARTS   AGE     IP            NODE           NOMINATED NODE   READINESS GATES
mysql   1/1     Running   0          7m30s   172.17.0.10   controlplane   [none]           [none]
web     1/1     Running   0          25m     172.17.0.4    controlplane   [none]           [none]

## Localziar los servicios del espacio de nombre payroll
kubectl get service -n payroll 
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
mysql         ClusterIP   172.20.204.219   [none]        3306/TCP   2m31s
web-service   ClusterIP   172.20.99.46     [none]        80/TCP     10m

## Corrige un fallo del fichero mysql_deploy.yaml, localiza las lineas afectadas, edita el deploy y applica la webapp
kubectl describe deployments.apps webapp | grep DB
      DB_Host:      mysql
      DB_User:      root
      DB_Password:  paswrd

kubectl get deployments.apps webapp -o yaml > deploy_webapp_file.yaml
nano deploy_webapp_file.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2026-07-27T19:57:26Z"
  generation: 1
  labels:
    name: webapp
  name: webapp
  namespace: default
  resourceVersion: "1937"
  uid: b466c08a-5c29-4b33-be6d-363fd9e89a4a
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: webapp
    spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql.payroll
        - name: DB_User
          value: root
        - name: DB_Password
          value: paswrd
        image: mmumshad/simple-webapp-mysql
        imagePullPolicy: Always
        name: simple-webapp-mysql
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
  - lastTransitionTime: "2026-07-27T19:57:31Z"
    lastUpdateTime: "2026-07-27T19:57:31Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-07-27T19:57:26Z"
    lastUpdateTime: "2026-07-27T19:57:31Z"
    message: ReplicaSet "webapp-57f9844586" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 1
  terminatingReplicas: 0
  updatedReplicas: 1
kubectl apply -f deploy_webapp_file.yaml 
Warning: resource deployments/webapp is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/webapp configured
Respuesta de la pagina_web de pruebas --> Environment Variables: DB_Host=mysql.payroll; DB_Database=Not Set; DB_User=root; DB_Password=paswrd;
From webapp-f8788bffc-466vz!

## Realizar una consulta nslooup al servidor mysql.payroll a traves de kubectl exec con el pod hr
kubectl exec hr -- nslookup mysql.payroll 
Server:         172.20.0.10
Address:        172.20.0.10#53

Name:   mysql.payroll.svc.cluster.local
Address: 172.20.116.14

## Crear un fichero con las respuestas del comando
kubectl exec hr -- nslookup mysql.payroll > /root/CKA/nslookup.out 

## Localizar el fichero
ls -l /root/CKA/
total 4
-rw-r--r-- 1 root root   0 Jul 27 20:15 nslookup.out

## Visualizar el fichero exitoso
cat /root/CKA/nslookup.out 
Server:         172.20.0.10
Address:        172.20.0.10#53

Name:   mysql.payroll.svc.cluster.local
Address: 172.20.204.219
