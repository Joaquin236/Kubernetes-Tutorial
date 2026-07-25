## 84.1º El contenedor de Docker conecta a la red con una interfaz virtual, el host recibe una conexión directa al contenedor para la interacción. Si el contenedor está iniciado en una red [none] estará totalmente aislado de la red
docker run --network none nginx
## Este contenedor no está enlazado con una red válida para su administración y despliegue de servicios. Tampoco puede enlazar con el contenedor vecino.

## 84.2º Si establecemos el contenedor con una red anfitrión si estará conectada, pero solo puede haber un contenedor escuchando el puerto de la conexión
docker run --network host nginx

## 84.3º Para establecer varios contenedores que escuchen un mismo puerto necesitamos usar el modo puente, creando una red privada que ofrezca este servicio
docker run nginx #--> ejecutar al menos dos veces
docker network ls
ip link add docker0 type bridge
ip addr
ip netns
docker inspect [container_ID]
ip -n [container_ID] link
## La interfaz del host lleva la IP 172.17.0.1/16, los contenedores llevan las direcciones posteriores a la serie de las IPs. Cada contenedor nuevo recibe un espacio de nombres y una nueva IP. Si usamos el comando curl y la url del contenedor devuelve el contenido de la página inicial
curl http://172.17.0.3:80 --> Welcome to nginx!
## Si este comando lo realizas desde un cliente que está fuera del entorno del contenedor debe dar un fallo de acceso a la url.

## 84.4º Docker ofrece la opción de publicar el puerto y ofrecer el acceso externo
docker run -p {8080}:80 nginx
## Para acceder necesitamos la IP del anfitrión y el puerto de reenvío:
curl http://[IP-HOST]:8080 #--> Welcome to nginx

## 84.5º Otra opción es crear una tabla de enrutamiento desde iptables
iptables -t nat -A PREROUTING -j DNAT --dport 8080 --to-destination 80
iptables -t nat -A DOCKER -j DNAT --dport 8080 --to-destination 172.17.0.3:80
## Consultamos la nueva tabla con el comando:
iptables -nvl -t nat

## 84.6º ["1-Crear_el_espacio_de_nombre","2-Crear_el_puente_de_red/interfaz","3-Crear_interfaz-virutal/tuberias","4-Enlazar_interfaz-virtual-al-espacio-de-nombre","5-Enlazar_otras_interfaces-virtuales_al_puente","6-Asignar_las_IP","7-Activar_interfaces","8-Activar-el-enmascaramiento-traductor-de-direcciones-IP"]. Estos pasos son válidos en: ["Linux","Docker","RKT","MESOS","Kubernetes"]
bridge add [Container_ID] /var/run/netns/[name_space_ID]

## 84.7º La interfaz de red de contenedores contiene las normas para definir el desarrollo de los programas y revolver incidencias en la red del contenedor. Los programas son los plugins de la infraestructura, el servicio CNI debe establecer la llamadas de los tiempos de ejecución de los contenedores. ["La_rutina_de_contenedores_debe_crear_el_espacio_de_nombre","Debe_identificar_la_red_a_la_que_conectará_el_contenedor","La-rutina-de-contenedor-llama-al-plugin-de-red_cuando_el_contenedor-","La-rutina-de-contenedor-llama-al-plugin-de-red"]
https://kubernetes.io/docs/concepts/cluster-administration/addons/ #--> Documentación sobre los plugins para Kubernetes
https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model # --> Documentación sobre el modelo de redes en Kubernetes
