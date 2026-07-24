## 82.1º El sistema de red se basa en las tablas de enrutamiento, puerta de enlace, la dirección del host, dirección de los contenedores y las subredes del switch/router. El DNS identifica un nombre con las direcciones de cada servidor para escribir el nombre canónico y evitar adivinar/memorizar la IP del recurso asociado. Los espacios de nombre y el docker también forman parte de la infraestructura. 

## 82.2º El conmutador enlaza los servidores con los clientes para crear una red de conexión manual, si un servidor tiene la opción de distribuir direcciones IP, ofrecerá las direcciones IP a cada dispositivo conectado. El conmutador solo puede transmitir a una sola red por defecto, si se admite puede crear redes virtuales y/o dividir en más de dos subrredes.

## 82.3º El comando para añadir una dirección IP es:
ip addr add ["Dirección_IP/CIDR"] dev eth0

## 82.4º Si necesitamos interactuar con otra red diferente que tiene una Dirección o CIDR, no es posible usar una conexión directa, necesitamos una tabla de enrutamiento ofrecita por el router. El router recoge las direcciones de una infraestructura y con la puerta de enlace distribuye los metadatos a cada red, permitiendo que un dispositivo cliente/servidor o cliente/cliente ubicado en distintas red/subred se comunique sin errores.

## 82.4º El comanto para visualizar la tabla de enrutamiento es:
route
## Para agregar una manualmente usamos el parámetro add, la dirección con la CIDR_MASK y la dirección del router
route add [Network_dir/CIDR] via [IP_Router/Gateway]
## Si volvemos a imprimir la tabla debemos observar la nueva tabla de enrutamiento
route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         DESKTOP-NDMMRA2 0.0.0.0         UG    0      0        0 eth0
172.20.112.0    0.0.0.0         255.255.240.0   U     0      0        0 eth0

## 82.5º Si necesitamos que el host con Linux realice un avance de IP, tenemos que localizar el fichero ip_forward:
cat /proc/sys/net/ipv4/ip_forward
0
## Si usamos comando echo, 1 y la redirección de salida, se podrá cambiar el valor y permitir la transmisión de metadatos a través del sistema:
echo 1 > /proc/sys/net/ipv4/ip_forward
1

## 83.1º