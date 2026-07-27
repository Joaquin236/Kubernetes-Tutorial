## 87.1 Cuando un se ha creado un servicio, los pods del cluser pueden comunicarse con la subred, el pod se aloja en el nodo y el servicio en el cluster y no se vincula con el nodo. Si un pod alojado en una aplicacion de bases de datos a la que accede desde el cluster, el servico esta funcionando bien. El nodo adyacente tiene un pod con una aplicacion web para dar accesibilidad desde fuera del nodo, para realizarlo necesitamos un puerto de nodos.

## 87.2 El puerto de nodos tambien tiene una direccion IP para acceder al cluster, los pods acceden al puerto de nodo, este servicio tiene el puerto 30080. 

## 87.3 Partiendo de un cluser de tres nodos sin pods, se esta ejecutando el kubelet en cada nodo, prepado para crear los pods. El kube-apiserver esta en el cluster y conectado a cada nodo, durante la creacion del pod le asigna la IP. El kubelet llama al kube-proxy, este servicio vigila los cambios en el cluster a traves del apiserver. Cada creacion de servicio se activa un proxy. Los servicios acceden por todo el cluster, no hay nignun servicio escuchando en la ip del servicio, no hay procesos, espacios de nombres, interacciones para un servicio.

## 87.4 Cuando creamos un objeto virtual, recibe una IP con rango definido, el kube-proxy crea las reglas de conexion y reenvio de puerto, el pod se le realiza un reenvio a la direccion del recurso accesible a cualquier nodo del cluster, en cada url se inserta la IP:Puerto para acceder a cada objeto que necesitemos consultar. En el proceso de crear/borrar los objetos virtuales tambien afecta a las reglas de conexion.

## 87.5 Kube-proxy admite diversas formas de administrar este sistema, el espacio de usuario. [El_kube-proxy_escucha_el_puerto_para_cada_servicio]. IPVS. [conexiones_a_los_pods_con_reglas_IPVS]. IPtables. [La_opcion_por_defecto].

## 87.6 Los comandos para usar el proxy son:
kube-proxy --proxy-mode [userspace]
kube-proxy --proxy-mode [iptable]
kube-proxy --proxy-mode [ipvs]

## 87.7 El pod detectado lleva la IP [10.244.1.2], ubicado en el nodo-1, el servicio lleva la IP [10.103.132.104], con el puerto [3306]. Ambos estan asociados a las bases de datos.
kubectl get pods -o wide
kubectl get service
kube-api-server --service-cluster-ip-range ipNet (Default: 10.0.0.0/24)
ps aux | grep kube-api-server
## La tabla IP es:
|IP:Port        |Fordward to|
|---------------|-----------|
|10.99.13.178:80|10.244.1.2 |
## La api del servicio establece el rango de direcciones con el comando. Al filtrar los procesos del comando ps aux con el servidor obtenemos la informacion.

## 87.8 Usando el comando iptables, localizamos las conexiones tcp del servicio db-service. Debe mostrar informacion asociada al db-service
iptables -L -T nat | grep db-service

## 87.9 El fichero /var/log/kube-proxy.log contiene el registro con los sucesos del kube-proxy
cat /var/log/kube.log
