## 88.1º Los objetivos del DNS en Kubernetes son: [Los_nombres_que_llevaran_asignados_los_objetos,Registrar_DNS_de_Servicios,Registrar_DNS_del_Pod]

## 88.2º Dentro del cluster hay tres nodos, cada uno lleva un nombre de dominio y la IP. Kubenertes despliega un DNS por defecto que configura el cluster.

## 88.3º El pod test lleva la ip 10.244.1.5, el web-service lleva la ip 10.107.37.188 y el pod web lleva la ip 10.244.2.5. Al llevar direcciones de red distintas estan alojadas en nodos distintos, para dar acceso desde el servidor web al servidor test a traves de un puerto, necesitamos el servicio web-service. Por cada servicio creado, el DNS de Kubernetes crea un registro del nombre y la ip.

## 88.4º El comando curl admite el uso de nombre de dominio, el espacio de nombre de esta infraestuctura es el Default.
curl http://web-service(.default)

## 88.5º Si el pod web y el servicio web-service se cambian al espacio de nombres apps, se necesita escribir
curl http://web-service.apps

## 88.6º Los servicios se agrupan en el subdominio SVC
curl http://web-service.apps.svc

## 88.7º Todos los objetos se agrupan en el dominio raiz del cluster, llamado cluster.local.
curl http://web-service.apps.svc.cluster.local
curl http://web-service.apps.pod.cluster.local

## 88.8º Los Pods no se les asigna este sistema por defecto, se puede habilitar, el sistema crea un nombre basado en la ip separado por guiones, el espacio de nombre es el mismo y el tipo es el pod, la raiz siempre sera la misma.
|HostName   |NameSpace|Type |Root         |IP Address   |
|-----------|---------|-----|-------------|-------------|
|web-service|apps     |SVC  |cluster.local|10.107.37.188|
|10-244-2-5 |apps     |pod  |cluster.local|10.244.2.5   |
|10-244-1-5 |default  |pod  |cluster.local|10.244.2.5   |

## 88.9º El pod test y el pod web siguen usando las mismas direcciones IP, si hacemos cat /etc/hosts debe aparecer la IP de cada Pod. El DNS tiene memorizado las IP en la una tabla. Los valores estan agregados en el fichero /etc/resolv.conf con el nameserver y la ip del servidor. La IP del DNS es: 10.96.0.10. Por cada Pod nuevo se agrega a la tabla de los nombres y la ip. Los pods necesitan su propia forma de realizar el registro en el DNS

## Tabla DNS normal
|Name-Pod|IP-Pod     |
|--------|-----------|
|web     |10.244.2.5 |
|test    |10.244.1.5 |
|db      |10.244.2.15|

## Tabla DNS con el nombre de IP
|Name-Pod   |IP-Pod     |
|-----------|-----------|
|10-244-2-5 |10.244.2.5 |
|10-244-1-5 |10.244.1.5 |
|10-244-2-15|10.244.2.15|

## 88.10º El CoreDNS se despliega como un Pod en el espacio de nombres kube-system en el cluster kubernetes, incluso tiene una replica para garantizar el acceso al servicio, esta creado a partir de una ReplicaSet. Este pod ejecuta el fichero ~/coredns, es el ejecutable que usamos cuando desplegamos un DNS. Depende de un fichero de nucleo /etc/coredns/Corefile. El fichero incorpora plugins para administrar y evaluar el servidor DNS, en este fichero se encuentra el plugin para que los Pods puedan tener el registro automatico pero esta desactivado, si se activa se crea automatico los registros.

## Linea del fichero a verificar
nano /etc/coredns/Corefile
.:53 {
    errors
    health
    kubernetes cluster.local in-addr.arps ip6.arps {
        pods insecure
        upstream
        fallthrough in-addr.arps ip6.arps
    }
    prometheus :9153
    proxy . /etc/resolv.com
    cache30
    reload
}

## 88.11º El fichero esta enlazado a un pod y lo interpreta como objeto [configmap]. Para editar el fichero se necesita editar el configmap
kubectl get configmap -n kubesystem

## Con el plugin de DNS_POD activo, verificamos si el cluster localiza los pod/servicios nuevos para agregarlos a la tabla, lo siguiente es que este registrado en el fichero /etc/resolv.conf. Al desplegar el plugin DNS crea un servicio para otros componentes dentro del cluster por defecto, la IP de este servicio se configura como servidor de nombres en los pods. La ip detectada en este comando, se redirige al resolv.conf
kubectl get service -n kube-system

## El fichero /var/lib/kubernetes/config.yaml contiene una entrada del clusterDNS con el valor de una IP y el Dominio --> cluster.local. Despues de la configuracion exitosa se resuelve los nombres y las IP para los objetos y componentes de la infraestructura.
|Servers                                           |Pods (Esta Columna no es valida)                 |
|--------------------------------------------------|-------------------------------------------------|
|curl http://web-service                           |curl http://web-service                          |
|curl http://web-service.default                   |curl http://web-service.default                  |
|curl http://web-service.default.svc               |curl http://web-service.default.pod              |
|curl http://web-service.default.svc.cluster.local |curl http://web-service.default.pod.cluster.local|

## Si usamos los comandos nslookup, dig o host para buscar manualmente el nombre de dominio y la IP, devuelve el nombre de dominio completo y la IP asignada al objeto. Este comando lo devuelve a partir del fichero /etc/resolv.conf que contiene esta informacion. Estos comandos solo funcionan para localizar los servicios [SVC_Objects], los Pods no pueden ser localizados.
nslooup >set type=A >web-service
dig A web-service
host web-service

## Hay un comando alternativo para localizar un pod con el comando host
host 10-244-2-5.default.pod.cluster.local
nslookup >set type=A >10-244-2-5.default.pod.cluster.local
dig A 10-244-2-5.default.pod.cluster.local
