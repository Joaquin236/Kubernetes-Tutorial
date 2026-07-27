## 88.1 Los objetivos del DNS en Kubernetes son: [Los_nombres_que_llevaran_asignados_los_objetos,Registrar_DNS_de_Servicios,Registrar_DNS_del_Pod]

## 88.2 Dentro del cluster hay tres nodos, cada uno lleva un nombre de dominio y la IP. Kubenertes despliega un DNS por defecto que configura el cluster.

## 88.3 El pod test lleva la ip 10.244.1.5, el web-service lleva la ip 10.107.37.188 y el pod web lleva la ip 10.244.2.5. Al llevar direcciones de red distintas estan alojadas en nodos distintos, para dar acceso desde el servidor web al servidor test a traves de un puerto, necesitamos el servicio web-service. Por cada servicio creado, el DNS de Kubernetes crea un registro del nombre y la ip.

## 88.4 El comando curl admite el uso de nombre de dominio, el espacio de nombre de esta infraestuctura es el Default.
curl http://web-service(.default)

## 88.5 Si el pod web y el servicio web-service se cambian al espacio de nombres apps, se necesita escribir
curl http://web-service.apps

## 88.6 Los servicios se agrupan en el subdominio SVC
curl http://web-service.apps.svc

## 88.7 Todos los objetos se agrupan en el dominio raiz del cluster, llamado cluster.local.
curl http://web-service.apps.svc.cluster.local

## 88.8 Los Pods no se les asigna este sistema por defecto, se puede habilitar, el sistema crea un nombre basado en la ip separado por guiones, el espacio de nombre es el mismo y el tipo es el pod, la raiz siempre sera la misma.
|HostName   |NameSpace|Type |Root         |IP Address   |
|-----------|---------|-----|-------------|-------------|
|web-service|apps     |SVC  |cluster.local|10.107.37.188|
|10-244-2-5 |apps     |pod  |cluster.local|10.244.2.5   |
|10-244-1-5 |default  |pod  |cluster.local|10.244.2.5   |

## 88.9 
