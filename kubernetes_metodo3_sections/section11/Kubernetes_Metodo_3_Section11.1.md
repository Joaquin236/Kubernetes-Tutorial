## 94.1º Para crear un cluster nuevo, necesitamos el kubeadm. Esta herramienta realiza las operaciones que en otras situaciones parecen ser más complicadas, configurando cada componente en orden. Después se inicia el cluster maestro. La subred virtual recibe los clientes.
Repositorio Certified_KodeKloud & vangrant --> https://github.com/kodekloudhub/certified-kubernetes-administrator-course
Documentación del Kubeadm --> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## 94.2º Para instalar el vangrant necesitamos clonar el repositorio, crea un fichero comprimido con el contenido. Extraemos lo que contiene y modificamos el fichero para asociarlo a la subred del solo-anfitrión del VirtualBox. Si está todo correcto crea una infraestructura que se puede consultar, añadir nodos, modificar y borrar.

## 94.3º A parte también se puede instalar el kubeadm, kubelet y kubectl para administrar desde el cluster hasta el contenedor. Para obtenerlo necesitamos añadir los repositios del apt de Linux, actualizar la distribución de Linux y usar el comando para instalar. Después instalamos el containerd, si es necesario creamos el directorio containerd
sudo mkdir -v /etc/containerd
containerd config default
containerd config default | sed 's/SystemdCgroup = false/SystemdCgroup = true/' sudo tee /etc/containerd/config.toml
cat /etc/containerd/config.toml
grep -i SystemdCgroup /etc/containerd/config.toml
sudo systemctl restart containerd
ip a
kubeadm init --apiserver-advertise-address 192.168.56.11 --pod-network-cidr "10.244.0.0/16" --upload-certs
sudo cat /etc/kubernetes/admin.conf
mkdir -vp $HOME/.kube
sudo cp -vi /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown -v $(id -u):$(id -g) $HOME/.kube/config
wget -vd --show-progress https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
nano kube-flannel.yml
kubectl apply -f kube-flannel.yml
kubectl get pods -n kube-flannel
kubeadm join 192.168.56.11:6443 --token ["token"] --discovery-token-ca-cert-hash sha256:["sha256"]
