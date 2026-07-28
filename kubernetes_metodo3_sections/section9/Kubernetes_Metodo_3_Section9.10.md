## 89.1 Ingress ofrece a los usuarios el acceso a su aplicacion a traves de una sola URL de acceso externo configurable para dirigir el trafico a diferentes servicios dentro del cluster, impantando la seguridad SSL. 

## 89.2 Si imaginamos el Ingress como un equilibrador de carga siete integrado en el cluster usando los objetos creadon en Kubernetes. El Ingress hay que establecerlo con el puerto de nodo o con un equilibrador de carga de la nube.

## 89.3 Empezamos con un proxy inverso o equilibrador de carga con nginx, traefik o haproxy. Kubernetes los configura para crear las tablas de enrutamiento de trafico a los servicios. Durante la configuracion se definen rutas URL y el SSL. El Ingress se implementa como una solucion compatible y se establece las reglas de configuracion. Los recursos se definen con ficheros de despliegue similares a los de los objetos. 

## 89.4 
