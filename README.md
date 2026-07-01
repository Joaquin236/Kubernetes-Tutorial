# IMPLEMENTACIÓN DE EJEMPLO. Método 1 --> 
Video_Tutorial de ejemplo --> https://www.youtube.com/watch?v=DzuU1KLQ240&t=902s

## 1º Descarga el k3s.io
curl -sfL https://get.k3s.io | bash - 

## 2.1º Descarga el kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

## 2.2º Instala el kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

## 2.3º Verifica el estado de kubectl
kubectl version --client --output=yaml 

## 3º Crea una plantilla de configuración, el contenido debe ser redirigido
sudo k3s kubectl config view --raw > ~/.config_k3s.yaml

## 4º Crea una variable con la ruta de fichero
export KUBECONFIG=~/.config_k3s.yaml 

## 5º Muestra todos los cluster activos
kubectl get node

## 6º Muestra los pods activos
kubectl get pods

## 7º Descarga el script que instala el helm, un gestor para kubernetes
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod +x -v get_helm.sh
./get_helm.sh

## 8º Muestra los comandos válidos para el helm
helm

# 9.1º Buscar packs para instalar (el enlace se muestra cortado con puntos suspensivos)
helm search hub podinfo

# 9.2º Instalar los charts
helm install my-podinfo oci://ghcr.io/stefanprodan/charts/podinfo

# 10.1º Muestra los valores del chart antes de instalar
helm show chart oci://ghcr.io/stefanprodan/charts/podinfo

# 10.2º Verificar la instalación del charts
kubectl port-forward svc/my-podinfo 9898:9898 ; curl http://localhost:9898

# 11º Listar las entradas chart activas
helm list

# 12º Mostrar información del pod
helm status my-podinfo

# 13º Entrar en el pod activo
kubectl -n default port-forward deploy/my-podinfo 8080:9898

## Conclusión: Es simple de usar pero no es extenso
------------------------------------------------------------------------------------------------------------------------------
# IMPLEMENTACIÓN AVAZADA. # ADVERTENCIA: SI HAS USADO EL EJEMPLO ANTERIOR DARÁ CONFLICTO CON EL EJEMPLO AVAZADO
## Documentación web del desarrollo -->
https://pabpereza.dev/docs/cursos/kubernetes/instalacion_de_kubernetes_cluster_completo_ubuntu_server_con_kubeadm
## Video_Tutorial de ejemplo. Método 2 -->
https://www.youtube.com/watch?v=eqxQGmem_bc&list=PLQhxXeq1oc2k9MFcKxqXy5GV4yy7wqSma
## Diseño de Script
# FASE_1: NODO MAESTRO
## 1º Actualizar repositorios, instalar certificados de repositorio y requisitos previos
apt update && apt upgrade -y
apt install curl apt-transport-https git wget software-properties-common lsb-release ca-certificates socat -y

## 2º Desactivar SWAP, este servicio de contenedores no admite SWAP activa
swapoff -a
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab # Auto comenta la línea de swap en fstab

## 3º Cargar los módulos del kernel y systemctl/sysctl
modprobe overlay
modprobe br_netfilter

cat << EOF | tee /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

## 4º Aplicar la configuración sysctl
sysctl --system

## 5º Instalar las claves GPG para instalar contend
mkdir -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
| sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

## 6º Instalar containerd y configurar el grupo necesario para el containerd
apt update && apt install containerd.io -y
containerd config default | tee /etc/containerd/config.toml
sed -e's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml
systemctl restart containerd

## 7º Instalar claves GPG de Kubernetes
mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

## 8º Añadir repositorio de kubernetes
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
apt update

## 9º Instalar kubeadm, kubectl kubelet
apt install -y kubeadm=1.33.1-1.1 kubelet=1.33.1-1.1 kubectl=1.33.1-1.1
apt-mark hold kubelet kubeadm kubectl # Bloquear actualizaciones automáticas

## 10º Buscamos nuestra IP y lo agregamos a nuestro fichero hosts para enlazar la IP de usuario con el cluster maestro
cat /etc/hosts
ip a | grep 192
echo "192.168.2.2 k8scp" >> /etc/hosts
cat /etc/hosts

## 11º Iniciar el cluster completo, es importante elegir el rango de IP para los pods.
## Plantilla comodin kubeadm init --pod-network-cidr=<rango de IPs para pods> --control-plane-endpoint=<nombre añadido en el /etc/hosts>:6443
## Comando de referencia:
kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=k8scp:6443 --v=9

## Si da conflicto el cluster del ejemplo_1 hacer los siguientes pasos;
## sudo /usr/local/bin/k3s-uninstall.sh # --> borrar el k3s que ha generado el conflicto y bloqueo de puertos
## sudo systemctl restart containerd # --> reiniciar el proceso containerd

## Volver a usar el comando inicial:
## kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=k8scp:6443 --v=9

## Si muestra este mensaje, lo has logrado:
## Your Kubernetes control-plane has initialized successfully!

## 12º Ejecutar los 5 comandos sugeridos por el comando exitoso
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
## Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf # --> echo $KUBECONFIG --> /etc/kubernetes/admin.conf

## You should now deploy a pod network to the cluster.
## Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
##  https://kubernetes.io/docs/concepts/cluster-administration/addons/

## You can now join any number of control-plane nodes by copying certificate authorities.
## And service account keys on each node and then running the following as root:

##  kubeadm join k8scp:6443 --token r2gs4v.lb1kt27wutj3yog1 \
##        --discovery-token-ca-cert-hash sha256:4fcb8aa36a49b664bd64071059bf6ab9132c9c1a41eee1a456f2e4d1a88976a3 \
##        --control-plane 

## Then you can join any number of worker nodes by running the following on each as root:

## kubeadm join k8scp:6443 --token r2gs4v.lb1kt27wutj3yog1 \
##        --discovery-token-ca-cert-hash sha256:4fcb8aa36a49b664bd64071059bf6ab9132c9c1a41eee1a456f2e4d1a88976a3 

## 13º Instalamos el helm
curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt update
sudo apt install helm -y

## 14º Instalar cilium, la aplicación de interfaz red de contenedores
helm repo add cilium https://helm.cilium.io/
helm repo update
helm template cilium cilium/cilium --version 1.17.4 \
--namespace kube-system > cilium.yaml
kubectl apply -f cilium.yaml

## 15º (Paso opcional y poco aconsejable) el nodo maestro también puede ejecutar acciones como si fuese un nodo trabajador
## pero su uso no es requerido. (Usar bajo responsabilidad). Si el cluster tiene un solo nodo se puede admitir.
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

## Para restablecer los cambios usaremos este comando;
## kubectl taint nodes --all node-role.kubernetes.io/control-plane:NoSchedule

# FASE_2: NODO TRABAJADOR
## Esta fase la realizaremos sobre el servidor que queremos añadir al cluster.

## Verificar el estado de la red
## curl -k https://k8scp:6443/version

## 1º Repetir los 10 primeros pasos de la FASE_1. Cuando se repita el paso 10 tenemos que añadir el nombre del cluster maestro en ejecución
kubectl get nodes
## cat /etc/hosts
ip a | grep 192
## echo "192.168.2.2 k8scp" >> /etc/hosts
cat /etc/hosts

## 2º Revisamos el token en curso con el comando:
kubeadm token list
## Este código tiene 24 horas de uso, necesitaremos crear otro cuando caduque.
# Creamos otro token con el comando:
# kubeadm token create --> comando normal
kubeadm token create --print-join-command # --> comando mejorado y crea el hash, con la unión completa
kubeadm join k8scp:6443 --token ypjplx.vbw2gftzyut98rby --discovery-token-ca-cert-hash sha256:5ec0d6b45244fc005c5fbb701930737ac0a49664f3263d243fcf124e1e3d3491 --v=9

## Otra fórmula de obtener el hash es;
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

## 3º Verificar la conexión con:
kubectl get nodes

## Conclusión: Tras realizar varios intentos de enlazar el nodo trabajador con el cluster. No se logró realizar con éxito este método
-----------------------------------------------------------------------------------------------------------------------------------
# IMPLEMENTACIÓN ALTERNATIVA
# Video de origen del Método 3 --> https://www.youtube.com/watch?v=DCoBcpOA7W4&t=168s
# Documentación 1: https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download
# Documentación 2: https://kubernetes.io/docs/concepts/services-networking/ingress/
# Documentación 3: https://kind.sigs.k8s.io
# Conclusión: Este método es el que más éxito tiene por ahora
# 1º Descargamos el minikube
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

# 2º Iniciamos el minikube, no usar el usuario '#', este comando solo funciona con '$'
minikube start

# 3º Accedemos al cluster, mostramos información y acceder al servidor con dashboard
kubectl get po -A
minikube dashboard

# 4.1º Crea una muestra de desarrollo y exponer en el puerto 8080:

kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
# Esto tomará un tiempo, pero el desarrollo mostrará cuando puedas iniciarlo:

kubectl get services hello-minikube
# La forma más fácil de acceder al servicio es iniciando el minikube con el navegador web:

minikube service hello-minikube
# Alternativamente se puede declarar el puerto:

kubectl port-forward service/hello-minikube 7080:8080
# Excelente, tu aplicación estará accesible con la URL http://localhost:7080/.

# Prodrás ver la respuesta de la aplicación en la salida de la consola. Intenta cambiar la ruta y observa los cambios.
# Puedes pasar un parámetro por POST y obervar la salida.

# 4.2º Para acceder al desarrollo del LoadBalancer, usa el comando 'minikube tunnel':

kubectl create deployment balanced --image=kicbase/echo-server:1.0
kubectl expose deployment balanced --type=LoadBalancer --port=8080
# En otra consola de comandos, inicia el tunnel para crear una ruta IP para el desarrollo del balanced:

minikube tunnel
# Para buscar la ruta IP, inicia este comando y verifica la columna de IP_Externa:

kubectl get services balanced
# Your deployment is now available at <EXTERNAL-IP>:8080

# 4.3º Inicia el complemento INGRESS:

minikube addons enable ingress
/*
The following example creates simple echo-server services and an Ingress object to route to these services.

kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
    - name: foo-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
    - port: 8080
---
kind: Pod
apiVersion: v1
metadata:
  name: bar-app
  labels:
    app: bar
spec:
  containers:
    - name: bar-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: bar-service
spec:
  selector:
    app: bar
  ports:
    - port: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - http:
        paths:
          - pathType: Prefix
            path: /foo
            backend:
              service:
                name: foo-service
                port:
                  number: 8080
          - pathType: Prefix
            path: /bar
            backend:
              service:
                name: bar-service
                port:
                  number: 8080
---
Apply the contents
*/

kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
# Espera a la dirección del kubernetes Ingress:

kubectl get ingress
/*
NAME              CLASS   HOSTS   ADDRESS          PORTS   AGE
example-ingress   nginx   *       <your_ip_here>   80      5m45s
Note for Docker Desktop Users:
To get ingress to work you’ll need to open a new terminal window and run minikube tunnel and in the following step use 127.0.0.1 in place of <ip_from_above>.

Now verify that the ingress works
*/

$ curl <ip_from_above>/foo
# Request served by foo-app
# ...

$ curl <ip_from_above>/bar
# Request served by bar-app
# ...

5º Pausar el minikube sin afectar a los servicios activos:
minikube pause

# Reanudar minikube en pausa:
minikube unpause

# Detener el cluster:
minikube stop

# Cambiar el límite de memoria (Requiere reinicio):
minikube config set memory 9001

# Explorar catalogo de complementos disponibles:
minikube addons list

# Crea un segundo cluster con una versión anterior:
minikube start -p aged --kubernetes-version=v1.34.0

# Borra todos los cluster activos:
minikube delete --all

# 6.1º Añadir tres nodos, (1 Nodo Maestro y 2 Nodos Trabajadores):
minikube start --nodes=3 --driver=docker

# 6.2º Añadir un nodo trabajdor:
minikube node add --worker

# 7.1º Añadir los pods de forma automática:
kubectl run test --image=nginx

# 7.2º Crear un pod a través de un fichero pod.yaml, para aplicarlo se enlaza el comando kubectl con el fichero:
nano pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test2
spec:
  containers:
  - name: web
    image: nginx

kubectl apply -f pod.yaml

# 8º Editamos el pod con kubectl edit pod <Nombre_pod>:
kubectl edit pod test

# 9º A través del parámetro -o wide que ofrece el visor de pods, ofrece más información.
kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
test    1/1     Running   0          108m   10.244.0.8   minikube   <none>           <none>
test2   1/1     Running   0          101m   10.244.0.9   minikube   <none>           <none>

kubectl get pods test -o yaml # --> ofrece descipciones con formato yaml
kubectl describe pod test # --> ofrece descipciones con formato amigable

# Si añadimos "<comando> | less" podemos navegar por la salida del comando

# 10º Con el parámetro logs verificamos el registro de estado del pod
kubectl logs test
kubectl logs test2

# 11.1º Borramos el pod test
kubectl delete pod test
pod "test" deleted from default namespace

# 11.2º Editamos el fichero pod.yaml anterior y cambiamos el nombre y el tipo de imagen en ejecución:
nano pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test 
spec:
  containers:
  - name: web
    image: nginx:alpine

kubectl apply -f pod.yaml 
pod/test created

# 11.3º Si editamos un pod existente y aplicamos debe aparecer como configurado;
nano pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - name: web
    image: nginx

kubectl apply -f pod.yaml 
pod/test configured

kubectl get nodes ; kubectl get pods
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   13h   v1.35.1
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (37s ago)   6m9s
test2   1/1     Running   0             155m

# 12.1º Para obtener los namespaces, usamos el comando:
kubectl get namespaces
NAME                   STATUS   AGE
default                Active   13h
kube-node-lease        Active   13h
kube-public            Active   13h
kube-system            Active   13h
kubernetes-dashboard   Active   13h
# Los namespaces default y/o creados manualmente son los que tenemos que modificar.
# Los namespaces creados por el sistema suelen llevar contenido crítico y deben permanecer sin cambios.

# 12.2º También se puede obtener los pods de cualquier namespace
kubectl --namespace kube-system get pods
NAME                               READY   STATUS    RESTARTS        AGE
coredns-7d764666f9-jr4pq           1/1     Running   1 (12h ago)     13h
etcd-minikube                      1/1     Running   1 (12h ago)     13h
kube-apiserver-minikube            1/1     Running   1 (12h ago)     13h
kube-controller-manager-minikube   1/1     Running   4 (3h11m ago)   13h
kube-proxy-7rxmx                   1/1     Running   1 (12h ago)     13h
kube-scheduler-minikube            1/1     Running   1 (12h ago)     13h
storage-provisioner                1/1     Running   3 (3h10m ago)   13h

# 12.3 Creamos un nuevo namespace donde podamos editar
kubectl create ns development
namespace/development created

# 12.4 Si el siguiente pod lo queremos llevar a otro namespace, necesitamos establecerlo durante la creación del pod:
kubectl -n development run nginx --image=nginx

kubectl -n development get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          29s

kubectl -n default get pods
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (41m ago)   46m
test2   1/1     Running   0             3h16m

## 12.5 Si necesitamos borrar el namespace, borrará todo lo que contiene
kubectl -n development get pods ; kubectl delete ns development ; kubectl -n development get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          30m
namespace "development" deleted
No resources found in development namespace.

## Al buscar el namespace borrado ya no se muestra
kubectl get namespaces
NAME                   STATUS   AGE
default                Active   14h
kube-node-lease        Active   14h
kube-public            Active   14h
kube-system            Active   14h
kubernetes-dashboard   Active   14h

## 12.6º Este comando mostrará la configuración del cluster activo
kubectl config view

## 12.7º Estos valores pueden definirse en el pod manualmente
kubectl config set-context minikube --namespace=development
Context "minikube" modified.

kubectl get pods
No resources found in development namespace.

kubectl get ns
NAME                   STATUS   AGE
default                Active   14h
development            Active   7m14s
kube-node-lease        Active   14h
kube-public            Active   14h
kube-system            Active   14h
kubernetes-dashboard   Active   14h

## Después de los cambios todo lo nuevo va a development
kubectl config view

kubectl -n default get pods
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (97m ago)   102m
test2   1/1     Running   0             4h11m
kubectl apply -f pod.yaml 
pod/test created

kubectl -n default get pods
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (97m ago)   103m
test2   1/1     Running   0             4h12m

kubectl -n development get pods
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          30s

## Para revertir cambios solo necesitamos el mismo comando con el valor inicial
kubectl config set-context minikube --namespace=default
Context "minikube" modified.

