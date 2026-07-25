## 85.1º A parte de conectar los nodos a la red, también deben conectar los pods que contienen el contenedor en su interior. Kubernetes no implementa por completo la estructura de conexión, el administrador tiene que realizar las acciones.

## 85.2º El modelo de conexión es:["Cada_pod_llevará_una_IP","Cada_pod_se_comunicará_con_los_pods_del_nodo","Cada_pod_se_comunicará_con_los_pods_de_otros_nodos_del_entorno_sin_usar_NAT"]

## 85.3º Para establecer la red de los pods también usaremos los comandos:
ip link add v-net-0 type bridge
ip addr add 192.168.15.5/24 dev v-net-0
ip link set veth-red nets red
ip -n red link set veth-red up
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5
ip link set dev v-net-0 up
ip link add veth-red type veth peer name veth-red-br
ip -n red addr add 192.168.15.1 dev veth-red
ip link set veth-red-br master v-net-0
iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE

## 85.4º En una infraestructura con red local [192.168.1.0/24], hay 3 nodos con red de host ["*.11","*.12","*.13"], cada nodo tiene un pod, el nodo ofrece una red de puente v-net-0, cada red virtual/puente lleva ["10.244.1.0/24","10.244.2.0/24","10.244.3.0/24"], los pods tienen la red de host de la red virtual, para interactuar con otro pod de otro nodo y otro puente virtual necesita una tabla de enrutamiento. Cada uno de de los pods necesita hacer el camino de ida y vuelta.

## 86.1º Para consultar la configuración de las interfaces de red del contenedor se realiza un listado a los directorios desde /opt/cni/bin y /etc/cni/net.d
ls -l /opt/cni/bin ; ls -l /etc/cni/net.d
## Dentro del fichero /etc/cni/net.d/10-bridge.conf encontraremos un fichero con el formato JSON sobre la configuración del CNI y los valores de parámetro