## 86.1º Las responsabilidades del plugin del CNI son: ["Debe_soportar_argumentos(ADD/DEL/CHECK)","Debe_soportar_parámetros_del_ID-contenor,nombres_de_espacio,redes/subredes_virtuales","Debe_adminsitrar_las_IP_asignadas_al_POD","Debe_devolver_resultados_en_formato_legible"]. 

## Mostrar el contenido del directorio /etc/cni/net.d/*
ls -l /etc/cni/net.d/
total 4
-rw-r--r-- 1 root root 292 Jul 26 13:26 10-flannel.conflist

## Consultar los pods activos
kubectl get pods -o wide 
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
backend    1/1     Running   0          2m45s   172.17.0.4   controlplane   [none]           [none]
frontend   1/1     Running   0          2m45s   172.17.0.5   controlplane   [none]           [none]

## Visualiza en la consola el contenido de la URL 172.17.0.4
kubectl exec -it frontend -- curl -m 5 172.17.0.4
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

## Visitar el github de flannel
https://github.com/flannel-io/flannel

## Borra el CNI de Flannel para sustituirlo por Calico CNI
kubectl delete daemonset -n kube-flannel kube-flannel-ds
kubectl delete cm kube-flannel-cfg -n kube-flannel
rm -v /etc/cni/net.d/10-flannel.conflist

## Documentación de Calico
https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises

## Crear tres recursos descargados en las urls marcadas
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.0/manifests/operator-crds.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.0/manifests/tigera-operator.yaml
curl https://raw.githubusercontent.com/projectcalico/calico/v3.31.0/manifests/custom-resources.yaml -O

## Edita el ficnero custom-resources.yaml, sustituye la dirección del cidr por la 172.17.0.0/16
nano custom-resources.yaml 
# This section includes base Calico installation configuration.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
      - name: default-ipv4-ippool
        blockSize: 26
        cidr: 172.17.0.0/16
        encapsulation: VXLANCrossSubnet
        natOutgoing: Enabled
        nodeSelector: all()

---
# This section configures the Calico API server.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}

---
# Configures the Calico Goldmane flow aggregator.
apiVersion: operator.tigera.io/v1
kind: Goldmane
metadata:
  name: default

---
# Configures the Calico Whisker observability UI.
apiVersion: operator.tigera.io/v1
kind: Whisker
metadata:
  name: default

## Aplica los cambios realizados en el fichero
kubectl apply -f custom-resources.yaml 
installation.operator.tigera.io/default created
apiserver.operator.tigera.io/default created
goldmane.operator.tigera.io/default created
whisker.operator.tigera.io/default created

## Consulta el estado de los pods usando watch y kubectl
watch kubectl get pods -A
Every 2.0s: kubectl get pods -A                                                     controlplane: Sun Jul 26 14:04:21 2026

NAMESPACE         NAME                                       READY   STATUS              RESTARTS   AGE
calico-system     calico-apiserver-9f854dd87-84hwj           0/1     ContainerCreating   0          29s
calico-system     calico-apiserver-9f854dd87-qqbsh           0/1     ContainerCreating   0          29s
calico-system     calico-kube-controllers-65968d8dc9-mwvgh   0/1     ContainerCreating   0          29s
calico-system     calico-node-984sv                          0/1     Running             0          29s
calico-system     calico-typha-7fdf55db65-h6zz2              1/1     Running             0          29s
calico-system     csi-node-driver-kfsr8                      2/2     Running             0          29s
calico-system     goldmane-c7cfb995d-f5lxf                   0/1     ContainerCreating   0          29s
calico-system     whisker-6778f54f55-jfz8g                   0/2     ContainerCreating   0          10s
default           backend                                    1/1     Running             0          28m
default           frontend                                   1/1     Running             0          28m
kube-system       coredns-6f6c7df987-6vvc9                   1/1     Running             0          37m
kube-system       coredns-6f6c7df987-jqt8q                   1/1     Running             0          37m
kube-system       etcd-controlplane                          1/1     Running             0          38m
kube-system       kube-apiserver-controlplane                1/1     Running             0          38m
kube-system       kube-controller-manager-controlplane       1/1     Running             0          38m
kube-system       kube-proxy-lk97k                           1/1     Running             0          37m
kube-system       kube-scheduler-controlplane                1/1     Running             0          38m
tigera-operator   tigera-operator-758cb79bc7-njgrs           1/1     Running             0          15m

## Consulta los pods del systema de calico usando watch y kubectl
watch kubectl get pods -n calico-system
Every 2.0s: kubectl get pods -n calico-system                                       controlplane: Sun Jul 26 14:05:00 2026

NAME                                       READY   STATUS    RESTARTS   AGE
calico-apiserver-9f854dd87-84hwj           1/1     Running   0          68s
calico-apiserver-9f854dd87-qqbsh           1/1     Running   0          68s
calico-kube-controllers-65968d8dc9-mwvgh   1/1     Running   0          68s
calico-node-984sv                          1/1     Running   0          68s
calico-typha-7fdf55db65-h6zz2              1/1     Running   0          68s
csi-node-driver-kfsr8                      2/2     Running   0          68s
goldmane-c7cfb995d-f5lxf                   0/1     Running   0          68s
whisker-6778f54f55-jfz8g                   2/2     Running   0          49s

## Consulta los pods activos
kubectl get pods -o wide 
NAME       READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
backend    1/1     Running   0          20s   172.17.49.72   controlplane   [none]           [none]
frontend   1/1     Running   0          21s   172.17.49.71   controlplane   [none]           [none]

## Intenta obtener el contenido de la web del servidor 172.17.49.72
kubectl exec -it frontend -- curl -m 5 172.17.49.72
curl: (28) Connection timed out after 5002 milliseconds
command terminated with exit code 28

## Mientras de usaba el Flannel se obtuvo acceso a la página web y al usar el Calico ya está bloqueado