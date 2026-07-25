## Consultar los nodos activos
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   14m   v1.35.0   10.244.96.19    [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    [none]          13m   v1.35.0   10.244.241.35   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Consultar las IP del sistema y filtrar por dirección 10.244.96.19/32
ip a | grep -A1 -B2 10.244
4: eth0@if205796: [BROADCAST,MULTICAST,UP,LOWER_UP] mtu 1450 qdisc noqueue state UP group default qlen 1000
    link/ether 2a:81:67:07:06:a8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.244.96.19/32 scope global eth0
       valid_lft forever preferred_lft forever

## Consultar los enlaces de las interfaces
ip link
1: lo: [LOOPBACK,UP,LOWER_UP] mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: tunl0@NONE: [NOARP] mtu 1480 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
4: eth0@if205796: [BROADCAST,MULTICAST,UP,LOWER_UP] mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 2a:81:67:07:06:a8 brd ff:ff:ff:ff:ff:ff link-netnsid 0

## Cambiar al node01 y consultar el enlace de las interfaces, después regresa al controlpanel
ssh node01 ; ip link
1: lo: [LOOPBACK,UP,LOWER_UP] mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: tunl0@NONE: [NOARP] mtu 1480 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
4: eth0@if205796: [BROADCAST,MULTICAST,UP,LOWER_UP] mtu 1450 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 2a:81:67:07:06:a8 brd ff:ff:ff:ff:ff:ff link-netnsid 0
exit

## Consultar los enlaces y filtrar por cni
ip link | grep cni
6: cni0: [BROADCAST,MULTICAST,UP,LOWER_UP] mtu 1400 qdisc noqueue state UP mode DEFAULT group default qlen 1000
7: vethe6467d8e@if3: [BROADCAST,MULTICAST,UP,LOWER_UP] mtu 1400 qdisc noqueue master cni0 state UP mode DEFAULT group default 
    link/ether ba:06:87:9a:39:92 brd ff:ff:ff:ff:ff:ff link-netns cni-d530291f-9d06-85d8-d3b2-f54cbcfc7076
8: vethf7ca3f3b@if3: [BROADCAST,MULTICAST,UP,LOWER_UP] mtu 1400 qdisc noqueue master cni0 state UP mode DEFAULT group default 
    link/ether e2:69:0c:bf:a6:52 brd ff:ff:ff:ff:ff:ff link-netns cni-bab174c2-e476-e356-269b-faf0e051d1f4

## Consultar la dirección física de la interfaz cni0
ip link show cni0 
6: cni0: [BROADCAST,MULTICAST,UP,LOWER_UP] mtu 1400 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 0e:89:39:a3:0a:4e brd ff:ff:ff:ff:ff:ff

## Mostrar las rutas de la interfaz
ip route show
default via 169.254.1.1 dev eth0 
169.254.1.1 dev eth0 scope link 
172.17.0.0/24 dev cni0 proto kernel scope link src 172.17.0.1 
172.17.1.0/24 via 172.17.1.0 dev flannel.1 onlink 

## Mostrar el puerto del proceso kube-scheduler
netstat -nltp | grep kube-sch
tcp        0      0 127.0.0.1:10259         0.0.0.0:*               LISTEN      3207/kube-scheduler

## Mostrar los puertos activos del proceso etcd y que lleve el número 2379 como número de puerto
netstat -anp | grep etcd | grep 2379
tcp        0      0 127.0.0.1:2379          0.0.0.0:*               LISTEN      3251/etcd           
tcp        0      0 10.244.96.19:2379       0.0.0.0:*               LISTEN      3251/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:47696         ESTABLISHED 3251/etcd           
tcp        0      0 127.0.0.1:2379          127.0.0.1:48360         ESTABLISHED 3251/etcd 
