## Verfica los firechos de ip_forward y de bridge-nf-call-iptables. Cada fichero debe estar en 1
cat /proc/sys/net/ipv4/ip_forward
0 --> 1
----------------
cat /proc/sys/net/bridge/bridge-nf-call-iptables 
0 --> 1

sudo echo "1" > /proc/sys/net/ipv4/ip_forward ; sudo echo "1" > /proc/sys/net/bridge/bridge-nf-call-iptables 
 
## Crea una cuenta de servicio, cluster-role, cluster-role-binding, volumen persistente y un pod
kubectl create serviceaccount pvviewer
kubectl create clusterrole pvviewer-role --resource=persistentvolumes --verb=list
kubectl create clusterrolebinding pvviewer-role-binding --clusterrole=pvviewer-role --serviceaccount=default:pvviewer

nano pvviewer.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pvviewer
  name: pvviewer
spec:
  containers:
  - image: redis
    name: pvviewer
  # Add service account name
  serviceAccountName: pvviewer
kubectl apply -f pvviewer.yaml
pod/pvviewer created

## Crea un storageclass
nano rancher-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: rancher-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: rancher.io/local-path
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
kubectl apply -f rancher-sc.yaml 
storageclass.storage.k8s.io/rancher-sc created

## Crea un configmap y edita el deploy del ns cm-namespace
kubectl create configmap app-config -n cm-namespace \
  --from-literal=ENV=production \
  --from-literal=LOG_LEVEL=info

kubectl set env deployment/cm-webapp -n cm-namespace \
  --from=configmap/app-config

## Crea una clase de prioridades desde el ns low-priority
kubectl create namespace low-priority
Error from server (AlreadyExists): namespaces "low-priority" already exists

kubectl config set-context --current --namespace low-priority 
Context "kubernetes-admin@kubernetes" modified.

nano pc.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 50000
globalDefault: false
description: "Low priority class"
kubectl apply -f pc.yaml

kubectl get pod lp-pod -n low-priority -o yaml

nano lp-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: lp-pod
  namespace: low-priority
spec:
  priorityClassName: low-priority
  containers:
  - name: nginx
    image: nginx
kubectl delete pod lp-pod -n low-priority
kubectl apply -f lp-pod.yaml

## Crea una politica de red para dar acceso a un pod. Important: Don't Alter Existing Objects! (default-deny NetworkPolicy must still exist)
kubectl config set-context --current --namespace default 
Context "kubernetes-admin@kubernetes" modified.

kubectl get netpol -n default
kubectl describe netpol default-deny

nano /root/ingress-to-nptest.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
kubectl apply -f ingress-to-nptest.yaml
kubectl run test-conn --image=busybox --restart=Never --rm -it -- wget -qO- -T 5 http://np-test-service

## Vuelve a editar la politica de red
nano /root/ingress-to-nptest.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-to-nptest
  namespace: default
spec:
  podSelector:
    matchLabels:
      run: np-test-1
  policyTypes:
  - Ingress
  ingress:
  - ports:
    - protocol: TCP
      port: 80
kubectl apply -f ingress-to-nptest.yaml
kubectl run test-conn --image=busybox --restart=Never --rm -it -- wget -qO- -T 5 http://np-test-service
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
pod "test-conn" deleted from default namespace

## El nodo node01 necesita mantenimiento
kubectl taint node node01 env_type=production:NoSchedule
kubectl run dev-redis --image=redis:alpine
kubectl get pods -o wide

nano pod-redis.yaml
apiVersion: v1
kind: Pod
metadata:
  name: prod-redis
spec:
  containers:
  - name: prod-redis
    image: redis:alpine
  tolerations:
  - effect: NoSchedule
    key: env_type
    operator: Equal
    value: production
kubectl apply -f pod-redis.yaml
kubectl get pods -o wide | grep prod-redis
prod-redis   0/1     ContainerCreating   0          0s      <none>        node01   <none>           <none>

## Verifica el pv app-pv
kubectl create namespace storage-ns
namespace/storage-ns created

kubectl config set-context --current --namespace storage-ns
Context "kubernetes-admin@kubernetes" modified.

kubectl describe pv app-pv
kubectl describe pvc app-pvc -n storage-ns

kubectl get pvc app-pvc -n storage-ns -o yaml > pvc.yaml

nano pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"app-pvc","namespace":"storage-ns"},"spec":{"accessModes":["ReadWriteOnce"],"resources":{"requests":{"storage":"10Gi"}}}}
  creationTimestamp: "2026-08-17T09:52:43Z"
  finalizers:
  - kubernetes.io/pvc-protection
  name: app-pvc
  namespace: storage-ns
  resourceVersion: "5726"
  uid: 69d017af-0d6b-45d6-8dee-c363aaf183f8
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: rancher-sc
  volumeMode: Filesystem
status:
  phase: Pending
kubectl apply -f pvc.yaml
persistentvolumeclaim/app-pvc created

kubectl delete pvc -n storage-ns app-pvc
kubectl apply -f pvc.yaml
kubectl get pvc app-pvc -n storage-ns

## Verifica el cluster-info del apiserver. Cambia el valor de puerto
Change the 9999 port to 6443 and run the below command to verify:
kubectl cluster-info --kubeconfig=/root/CKA/super.kubeconfig

## El fichero solo necesita un cambio en una sola línea. No hay que sustituir todo
nano /root/CKA/super.kubeconfig
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: [base64_code]
    server: https://controlplane:9999 # Cambiar el puerto erroneo por --> 6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: storage-ns
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
users:
- name: kubernetes-admin
  user:
    client-certificate-data: [base64_code]
    client-key-data: [base64_code]

## Escalar el deploy para hacerlo más amplio
kubectl config set-context --current --namespace default 
Context "kubernetes-admin@kubernetes" modified.

kubectl scale deploy nginx-deploy --replicas=3
kubectl get pods -n kube-system

## Repara el nombre binario del manifest file
sed -i 's/kube-contro1ler-manager/kube-controller-manager/g' /etc/kubernetes/manifests/kube-controller-manager.yaml
kubectl get pods -n kube-system | grep controller-manager

kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           6m2s

## Crea un pod de escalada horizontal
nano api-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 1
  maxReplicas: 20
  metrics:
  - type: Pods
    pods:
      metric:
        name: requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
kubectl apply -f api-hpa.yaml
horizontalpodautoscaler.autoscaling/api-hpa created

## Crea un httpRoute para web-route y sus servicios
kubectl create -n default -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: web-route
  namespace: default
spec:
  parentRefs:
    - name: web-gateway
      namespace: default
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: web-service
          port: 80
          weight: 80
        - name: web-service-v2
          port: 80
          weight: 20
EOF
httproute.gateway.networking.k8s.io/web-route created

## Con el Helm instala y borra un chart
helm ls -n default
cd /root/
helm lint ./new-version
helm install webpage-server-02 ./new-version
helm uninstall webpage-server-01 -n default

## Corrige el CNI Plugin, el pod de la red CIDR necesita confirmar el cluster, el resultado se redirige al fichero pod-cidr.txt
kubectl -n kube-system get configmap kubeadm-config -o yaml | grep podSubnet
# podSubnet: 172.17.0.0/16

kubectl -n kube-system get configmap kubeadm-config -o yaml \
  | awk '/podSubnet:/{print $2}' > /root/pod-cidr.txt

cat /root/pod-cidr.txt
# 172.17.0.0/16

## Be careful not to confuse Cluster PodCIDR vs Node PodCIDR:
- Cluster = Entire pool
- Cluster PodCIDR (big range: 172.17.0.0/16)
   - Find it with : kubectl get cm kubeadm-config -n kube-system -o yaml
- Node = Single slice
- Node PodCIDR (small slice: 172.17.0.0/24)
   - Find it with : kubectl get node <name> -o jsonpath='{.spec.podCIDR}'  
