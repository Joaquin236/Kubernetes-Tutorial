## 118.1º La estructura del cluster tiene estas fases
## 118.2º Fase_1: Cluster de Kubernetes, la base donde se aloja el controlador, el DNS y el nodo.
## 118.3º Fase_2: Controlplane, aloja el servicio DNS para ofrecer nombres asociados a la IP de cada componente.
## 118.4º Fase_3: Nodos, contienen el proxy, servicios, pods/contenedor y plugins

## 118.5º Si usamos -l al comando kubectl get obtenemos información extra del pod
kubectl get pods -l app=hostmanes

## Si usamos -o al comando kubectl get se puede pasar una salida de objeto para recibir detalles concretos
kubectl get pods -l app=hostnames -o=jsonpath='{.items[*].status.podIP}'

## Si usamo este comando, obtenemos un código de bucle con direcciones IP
kubectl run -it --rm --restart=Never busybox --image=busybox sh

## Estos comandos mejoran la depuración de la estructura

## Extraemos el yaml de un servicio con este comando
kubectl get svc hostmanes -o yaml

## Consultamos los endpoint del ns marcado
kubeclt get endpoints -l kubernetes.io/service-name=hostnames -n default

## 118.6º La estructura del kube-system es
## Fase_1: El nodo principal
## Fase_2: Configuración: Config-map & servicio_cuenta
## Fase_3: Carga de trabajo, deploys & CoreDNS
## Fase_4: Nodos de redes y acceso a la red

