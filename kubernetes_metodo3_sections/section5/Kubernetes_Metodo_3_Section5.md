## 44.1º Los despliegues ofrecen las opciones de rolling update y rollback para administrar cambios y versionar los despliegues y ofrecer un sistema dinámico.
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment

## 44.2º Existen cinco formas de realizar cambios a un despliegue que esté completo:

## 1º Borrar el despliegue --> modificar el fichero.yaml --> volver a desplegar. Se conoce como Recreate, causa inaccesibilidad a los ususarios.

## 2º Sustituir los pods uno a uno --> retirar los pods antuguos consecutivos y poner los más nuevos, si se hace bien, la aplicación no debe notar la ausencia del servidor ni los usuaros se quedan sin conexión. Se conoce como Rolling update.

## Por defecto se usa la estrategia de Rolling Update:

## Plantilla del fichero.yaml:
nano deployment-definition.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: myapp-deployment
 labels:
     app: myapp
     type: front-end
spec:
  template:
     metadata:
      name: myapp-pod
      labels:
         app: myapp
         type: front-end
      spec:
        containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector:
     matchLabels:
        type: front-end

## Comando para crear el deploy:
kubectl create -f deployment-definition.yaml

## Comando para consultar el deploy:
kubectl get deployments

## Comando para actualizar:
kubectl apply -f deployment.yaml # --> requiere modificar fichero
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1 # --> comando instantaneo, el fichero no recibe este cambio

## Comando para consultar los cambios:
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment

## Deshacer los cambios fallidos:
kubectl rollout undo deployment/myapp-deployment

## Comando para actualizar un deploy:
kubectl set image deployments frontend simple-webapp=kodekloud/webapp-color:v2

## Comando para consultar el tipo de actualización del deploy:
kubectl describe deployments.apps frontend | grep StrategyType

## 45º Comparación entre un contenedor docker y un pod de kubernetes: Las opciones de arranque de los contenedores y pods son similares porque admiten los mismos comandos y argumentos. En docker es una ejecución individual mientras que en kubernetes si se ha usado deployment es una ejecución en grupo.

## FROM --> Ubuntu
## ENTRYPOINT --> ["sleep"]
## CMD --> ["5"]

## CMD: command param1 --> ["command","param1"]
## CMD: sleep 5 --> ["sleep","5"]

## Muestra de comando docker:
docker build -t ubuntu-sleeper
#
docker run ubuntu-sleeper
#
docker run ubuntu sleep 5
#
docker run --name ubuntu-sleeper ubuntu-sleeper 10
#
docker run --name ubuntu-sleeper \
-- entrypoint sleep2.0
ubuntu-sleep 10

# Estos contenedores al arrancar lleva el comando sleep para pausar la consola, el argumento son los segundos en pausa.

## Muestra de comando pod para kubernetes:
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args:  ["10"]

## Es fichero contiene el mismo contenedor con el diseño de kubernetes:

## Visalizar los comandos asociados a un pod:
kubectl describe pods ubuntu-sleeper | grep -A 3 Command
    Command:
      sleep
      4800
    State:          Running

## Crear un pod con un ubuntu que lleve el comando sleep 5000:
nano ubuntu-pod-sleeper2.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-2
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args:  ["5000"]

## Crear un pod que ofrecza un sitio web y comando python app.py --color green:
nano webapp-color-pod-USER.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["python", "app.py"]
    args: ["--color", "green"]

## 46º Las variables de entorno permiten modificar ajustes locales de los contenedores y pods.

## Variable de entorno en contenedor Docker:
docker run -e APP_COLOR=pink simple-weapp-color

## Variable de entorno en kubernetes:
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    env:
     - name: APP_COLOR
       value: pink

## En kubernetes usamos la entrada 'env' con el nombre_variable y valor. En docker usamos la opción -e nombre_variable y valor. Kubernetes ofrece más opciones para los env como ["Plain_Key_Value","ConfigMap","Secrets"]

## Entorno_1: Plain_key_value
env:
  - name: APP_COLOR
    value: pink
## Entorno_2: ConfigMap
env:
  - name: APP_COLOR
    valueFrom:
       configMapKeyRef:
## Entorno_3: Secrets
env:
  - name: APP_COLOR
    valueFrom:
       secretKeyRef:

## 47.1º En los casos que tenemos muchos ficheros para trabajar se vuelve difícil administrar el sistema, una de las opciones disponibles es el ConfigMap.

## 47.2º El configmap va por separado del fichero y/o el comando de crear el pod. El configmap también se puede establecer por comando y/o fichero.

## Crear el configmap con comando:
kubectl create configmap ["config-name"] \
--from-literal=["key"]=["value"]
##
kubectl create configmap app-config \
--from-literal=APP_COLOR=blue \
--from-literal=APP_MODE=prod
##
kubectl create configmap ["config-name"] \
--from-file=["/path/to/file.yaml"]
##
kubectl create configmap app-config \
--from-file=app_config.properties

## 47.3º Para establecer el configmap a través de fichero, primero se crea los ficheros con el pod y el valor del configmap:
## Crear el pod para asociar al fichero configmap:
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - configMapRef:
           name: app-config # this name is same to name of configmap_app-config.yaml

## Crear el configmap con fichero:
nano configmap_app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app_config # this name is some to name of pod-definition.yaml
data:
  APP_COLOR: blue
  APP_MODE: prod

nano configmap_mysql-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql_config
data:
  port: 3306
  max_allowed_packet: 128M

nano configmap_redis-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app_config
data:
  port: 6379
  rdb-compression: yes

## El comando create de kubectl se asocia al fichero:
kubectl create -f pod-definition.yaml # --> creamos el pod
kubectl create -f configmap.yaml # --> creamos el configmap

## Para consultar los configmaps disponemos de los comandos:
kubectl get configmaps
kubectl describe configmaps

## Consultar la variable de entorno con kubectl describe y filtar por entorno:
kubectl describe pods webapp-color | grep -A 4 env*

## Crea un pod temporal con la variable de entorno:
nano pod_webapp-color.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      valueFrom:
       configMapKeyRef:
         name: webapp-config-map
         key: APP_COLOR
    image: kodekloud/webapp-color
    name: webapp-color

## Crea un configmap para asociarlo a un pod:
kubectl create configmap webapp-config-map \
--from-literal=APP_COLOR=darkblue \
--from-literal=APP_OTHER=disregard

## Crea un nuevo pod para sustituir el pod anterior y asociarlo con el configmap:
nano pod_configmap-webapp-color.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      valueFrom:
       configMapKeyRef:
         name: webapp-config-map
         key: APP_COLOR
    image: kodekloud/webapp-color
    name: webapp-color
