## 42.1º La monitorización facilita la supervisión del nodo, los pods y contenedores, los recursos que se pueden monitoriar son: CPU, RAM & HDD/SSD. Disponemos de un catágolo de herramientas de dominio público para monitorizar el sistema. Metric Server, Elastic stack, Prometheus, DataDog & Dinatrace.

## 42.2º Heapster fue un proyecto inicial de suervisión de kubernetes que abrió el camino a otras herramientas de monitor de sistemas. La herramienta pionera Heapster ha sido descatalogada, ahora se necesita usar Metric Server.

## 42.3º Metric Server recoge los datos de uso de la CPU, RAM & HDD/SSD en uso pero no almacena un historial, todo se recoge en la RAM y no guarda en Disco. CAdvisor es una dependencia que trabaja en conjunto con el kubelet para recopilar en memoria las métricas.

## 42.4º Si estamos usando minikube se puede activar el complemento para usar el Metric Server con el comando:
minikube addons enable metrics-server

## 42.5º Si la estructura de kubernetes está montada con otra herramienta distinta:
git clone https://github.com/kubernetes-sigs/metrics-server.git
kubectl create -f deploy/1.8+

## 42.6º Una vez instalado usamos el comando:
kubectl top node

## 42.7º También tenemos el comando para monitorizar los pods:
kubuectl top pod

## Decargar el metric server para supervisar el sistema:
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

## 43.1º En Docker se puede desplegar un simulador de eventos para evaluar el modo de recopilar los eventos:
docker run -d kodekloud/event-simulator

## 43.2º Los eventos los guarda en el sistema para verlos con el comando;
dokcer logs -f ecf

## 43.3º Desde kubernetes se puede realizar la misma simulación a partir de un pod, el comando de aplicar y comando de los registros:

## 1º Creamos el pod nuevo:
nano event-simulator.yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator

## 2º Crear el pod con el fichero
kubectl apply -f event-simulator.yaml

## 3º Verificar los eventos
kubectl logs -f event-simulator-pod

## 4º Si añadimos una segunda imagen al contenedor:
nano event-simulator.yaml
apiVersion: v1
kind: Pod
metadata:
  name: event-simulator-pod
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
  - name: image-processor
    image: some-image-processor

## 5º Modificamos el comando de registros, este cambio implica escribir el nombre de la imagen que se quiere evaluar, el pod hay que actualizarlo con los nuevos valores:
kubectl logs -f event-simulator-pod event-simulator

## Conusltar el registro de un pod cuyo usuario esté bloqueado
kubectl logs webapp-1 | grep USER5

## Consultar varios errores de usuarios en un pod
kubectl logs webapp-2 | grep USER1 ; kubectl logs webapp-2 | grep USER30 ; kubectl logs webapp-2 | grep USER4 ; kubectl logs webapp-2 | grep USER2
