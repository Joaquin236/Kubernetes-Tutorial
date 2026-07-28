## 87.1º Cuando un se ha creado un servicio, los pods del cluser pueden comunicarse con la subred, el pod se aloja en el nodo y el servicio en el cluster y no se vincula con el nodo. Si un pod alojado en una aplicacion de bases de datos a la que accede desde el cluster, el servico esta funcionando bien. El nodo adyacente tiene un pod con una aplicacion web para dar accesibilidad desde fuera del nodo, para realizarlo necesitamos un puerto de nodos.

## 87.2º El puerto de nodos tambien tiene una direccion IP para acceder al cluster, los pods acceden al puerto de nodo, este servicio tiene el puerto 30080. 

## 87.3º Partiendo de un cluser de tres nodos sin pods, se esta ejecutando el kubelet en cada nodo, prepado para crear los pods. El kube-apiserver esta en el cluster y conectado a cada nodo, durante la creacion del pod le asigna la IP. El kubelet llama al kube-proxy, este servicio vigila los cambios en el cluster a traves del apiserver. Cada creacion de servicio se activa un proxy. Los servicios acceden por todo el cluster, no hay nignun servicio escuchando en la ip del servicio, no hay procesos, espacios de nombres, interacciones para un servicio.

## 87.4º Cuando creamos un objeto virtual, recibe una IP con rango definido, el kube-proxy crea las reglas de conexion y reenvio de puerto, el pod se le realiza un reenvio a la direccion del recurso accesible a cualquier nodo del cluster, en cada url se inserta la IP:Puerto para acceder a cada objeto que necesitemos consultar. En el proceso de crear/borrar los objetos virtuales tambien afecta a las reglas de conexion.

## 87.5º Kube-proxy admite diversas formas de administrar este sistema, el espacio de usuario. [El_kube-proxy_escucha_el_puerto_para_cada_servicio]. IPVS. [conexiones_a_los_pods_con_reglas_IPVS]. IPtables. [La_opcion_por_defecto].

## 87.6 Los comandos para usar el proxy son:
kube-proxy --proxy-mode [userspace]
kube-proxy --proxy-mode [iptable]
kube-proxy --proxy-mode [ipvs]

## 87.7º El pod detectado lleva la IP [10.244.1.2], ubicado en el nodo-1, el servicio lleva la IP [10.103.132.104], con el puerto [3306]. Ambos estan asociados a las bases de datos.
kubectl get pods -o wide
kubectl get service
kube-api-server --service-cluster-ip-range ipNet (Default: 10.0.0.0/24)
ps aux | grep kube-api-server
## La tabla IP es:
|IP:Port        |Fordward to|
|---------------|-----------|
|10.99.13.178:80|10.244.1.2 |
## La api del servicio establece el rango de direcciones con el comando. Al filtrar los procesos del comando ps aux con el servidor obtenemos la informacion.

## 87.8º Usando el comando iptables, localizamos las conexiones tcp del servicio db-service. Debe mostrar informacion asociada al db-service
iptables -L -T nat | grep db-service

## 87.9º El fichero /var/log/kube-proxy.log contiene el registro con los sucesos del kube-proxy
cat /var/log/kube.log

## Localizar el nodo controlplane
kubectl get nodes -o wide | grep control
controlplane   Ready    control-plane   15m   v1.35.0   10.244.102.214   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Filtrar el fichero /etc/kubernetes/manifest/kube-controller-manager.yaml para localizar el cidr
cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep 172.17
    - --cluster-cidr=172.17.0.0/16

## Consultar el servicio kubernetes
kubectl get service -o wide 
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   SELECTOR
kubernetes   ClusterIP   172.20.0.1   [none]        443/TCP   18m   [none]

## Consultar los pods del kube-system y filtrar por nombre kube-proxy* 
kubectl get pods -o wide -n kube-system | grep kube-proxy
kube-proxy-9rq62                           1/1     Running   0          21m   10.244.102.214   controlplane   [none]           [none]
kube-proxy-cdgqp                           1/1     Running   0          21m   10.244.34.228    node01         [none]           [none]

## Mostrar el registro de eventos del pod kube-proxy-cdgqp en kube-system
kubectl logs -n kube-system kube-proxy-cdgqp 
I0727 13:48:04.055113      [1_server_linux.go:53] "Using iptables proxy"
I0727 13:48:04.134124      [1_shared_informer.go:370] "Waiting for caches to sync"
I0727 13:48:04.234433      [1_shared_informer.go:377] "Caches are synced"
I0727 13:48:04.234460      [1_server.go:218] "Successfully retrieved NodeIPs" NodeIPs=["10.244.34.228"]

## desbribir el pod kube-proxy-9rq62
kubectl describe pods -n kube-system kube-proxy-9rq62

## Consultar el ds del espacio de nombres kube-system
kubectl get ds -n kube-system 
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
canal        2         2         2       2            2           kubernetes.io/os=linux   30m
kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   30m
