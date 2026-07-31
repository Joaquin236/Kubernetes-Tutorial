## Activar el avance de paquetes IPv4
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward=1
EOF

#Apply sysctl params without reboot

sudo sysctl --system

#verify that net.ipv4.ip_forward is set to 1 with:
sysctl net.ipv4.ip_forward

## Ejecutar los comandos para instalar el kubelet, kubeadm y kubectl
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

# To see the new version labels
sudo apt-cache madison kubeadm

sudo apt-get install -y kubelet=1.35.0-1.1 kubeadm=1.35.0-1.1 kubectl=1.35.0-1.1

sudo apt-mark hold kubelet kubeadm kubectl

## La versión instalada es la 1.35.0
kubectl get nodes
E0731 15:57:21.852776   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.853811   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.854630   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.856625   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.857438   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
Error from server (NotFound): the server could not find the requested resource

## Inicia el api server con el kubeadm
IP_ADDR=$(ip addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
kubeadm init --apiserver-cert-extra-sans=controlplane --apiserver-advertise-address $IP_ADDR --pod-network-cidr=172.17.0.0/16 --service-cidr=172.20.0.0/16

## Ejecuta los comandos para crear $HOME.kube/config
mkdir -pv $HOME/.kube
sudo cp -vi /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown -v $(id -u):$(id -g) $HOME/.kube/config

## Antes de acabar se ejecuta este comando
kubeadm join 10.244.253.204:6443 --token xd36qx.s9c7fj3ej4erwyec \
        --discovery-token-ca-cert-hash sha256:bd9d12ff81fd7fffab77040eb443dd5a4ac43e07eb269a8dab87a52ce7d01303

## Consultar la IP del servidor
ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
4: eth0@if190562: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 52:5b:da:a8:eb:c3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.253.204/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::505b:daff:fea8:ebc3/64 scope link 
       valid_lft forever preferred_lft forever
5: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UNKNOWN group default 
    link/ether 92:16:b7:7b:ed:6d brd ff:ff:ff:ff:ff:ff
    inet 172.17.1.0/32 scope global flannel.1
       valid_lft forever preferred_lft forever
    inet6 fe80::9016:b7ff:fe7b:ed6d/64 scope link 
       valid_lft forever preferred_lft forever


## descarga el kube-flannel
wget -vd --show-progress https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

## Edita el fichero para adaptarlo
## Sección de la subred
    {
      "Network": "10.244.0.0/16",
      "EnableNFTables": false,
      "Backend": {
        "Type": "vxlan"
      }
## Sección del argumento
      - args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth0 
## Aplicar cambios
kubectl apply -f kube-flannel.yml 
namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

## Consultar los pods de todos los espacios de nombres
kubectl get pods -A                                                                node01: Fri Jul 31 16:36:14 2026

NAMESPACE      NAME                             READY   STATUS    RESTARTS      AGE
kube-flannel   kube-flannel-ds-5qxqb            0/1     Error     4 (56s ago)   110s
kube-system    coredns-7d764666f9-n2x66         1/1     Running   0             22m
kube-system    coredns-7d764666f9-x2htp         1/1     Running   0             22m
kube-system    etcd-node01                      1/1     Running   0             22m
kube-system    kube-apiserver-node01            1/1     Running   0             22m
kube-system    kube-controller-manager-node01   1/1     Running   0             22m
kube-system    kube-proxy-mgqr7                 1/1     Running   0             22m
kube-system    kube-scheduler-node01            1/1     Running   0             22m

## Consultar los nodos
kubectl get nodes -o wide
NAME     STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
node01   Ready    control-plane   23m   v1.35.0   10.244.253.204   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26
