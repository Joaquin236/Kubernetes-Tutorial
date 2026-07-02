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
