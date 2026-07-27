## Consultar los pods, enfocarse en kube-system y filtrar por core
kubectl get pods -o wide -n kube-system | grep core
coredns-6f6c7df987-4rpj2               1/1     Running   0          12m   172.17.0.3      controlplane   [none]           [none]
coredns-6f6c7df987-sjlhj               1/1     Running   0          12m   172.17.0.2      controlplane   [none]           [none]

## Consultar los servicios, enfocarse en kube-system
kubectl get service -o wide -n kube-system 
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
kube-dns   ClusterIP   172.20.0.10   [none]        53/UDP,53/TCP,9153/TCP   14m   k8s-app=kube-dns

## Consultar el deploy de kube-system
kubectl get deployments.apps -n kube-system -o wide 
NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
coredns   2/2     2            2           17m   coredns      registry.k8s.io/coredns/coredns:v1.10.1   k8s-app=kube-dns

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

## Consultar los servicios en default
kubectl get service -o wide 
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE   SELECTOR
kubernetes     ClusterIP   172.20.0.1       [none]        443/TCP        23m   [none]
test-service   NodePort    172.20.161.141   [none]        80:30080/TCP   13m   name=test
web-service    ClusterIP   172.20.47.241    [none]        80/TCP         13m   name=hr

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

## Corrige un fallo del fichero mysql_deploy.yaml
