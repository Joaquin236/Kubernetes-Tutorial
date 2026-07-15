## 48.1º Los ficheros que estamos usando están en texto plano y pueden ser consultados por cualquiera, al suceder esto en caso de que puede ser fitrado datos sensibles, habrá problemas con la confidencialidad, integridad de credenciales y habrá que modificar las credenciales de urgencia. Con el sistema de los secretos codificamos los datos sensibles, también se puede migrar los datos al configmap pero no es el fichero apropiado para las contraseñas. Los secretos también tienen dos formas de ser creados: ["Comandos","Ficheros"]

## 48.2º Los valores que necesitamos codificar, encriptar o esconder digitalmente suelen ser:
["DB_HOST","mysql"]       # --> Host del sistema de la DB
["DB_User","root"]        # --> Usuario de la DB
["DB_password","passwd"]  # --> Contraseña de la DB

## Muestra de secreto creado por comando:
kubectl create secret generic ["secret-name"] \
--from-literal=["key"]=["value"]  \
--from-literal=["key"]=["value"] \
--from-literal=["key"]=["value"]
#
kubectl create secret generic app_secret \
--from-literal=DB_Host=mysql \
--from-literal=DB_User=root \
--from-literal=DB_Passwd=passwd

## Si se añade muchos campos se necesitará sustituir las entradas --from-literal=["key"]=["value"] se cambia por un fichero:
kubectl create secret generic app_secret \
--from-file=["/path/to/file.yaml"]

## También se puede localizar el campo específico para cambiar las propiedades:
kubectl create secret generic app_secret \
--from-file=app_secret.properties

## Muestra de secreto desde fichero.yaml:
nano secret-data.yaml # -->  Contiene los datos que necesitamos codificar.
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: mysql
  DB_User: root
  DB_Password: passwd

# Comando para crear secreto con el fichero
kubectl create -f ["secret-data.yaml"]

## El fichero sigue siendo un texto plano legible pero el sistema codifica los datos a través de base64.
echo "1010101" | base64 # --> Secuencia binaria en bruto
MTAxMDEwMQo= # --> Secuencia binaria codificada
#
echo "MTAxMDEwMQo=" | base64 --decode # --> Secuencia binaria codificada para decodificar
1010101 # --> Secuencia binaria en bruto de nuevo

## El sistema recolecta los tres valores y los devuelve con esta apariencia
echo "mysql" | base64
bXlzcWwK
echo "root" | base64
cm9vdAo=
echo "passwd" | base64
cGFzc3dkCg==

## Consultamos el estado de los secrets en:
kubectl get secrets
kubectl describe secrets
kubectl get secrets ["name_secret"] -o yaml

## El sistema devuelve los valores originales con la opción --decode que ofrece base64
echo "bXlzcWwK" | base64 --decode
mysql
echo "cm9vdAo=" | base64 --decode
root
echo "cGFzc3dkCg==" | base64 --decode
passwd

## Fichero codificado:
nano secret-data.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWwK
  DB_User: cm9vdAo=
  DB_Password: cGFzc3dkCg==

## Con el secrect creado agregamos el fichero con el pod:
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
labels:
  name: simple-webapp-color
spec:
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    envFrom:
      - secretRef:
          name: app-secret

## Con el comando de aplicar/crear definimos el contenido del pod:
kubectl create -f pod-definition.yaml

## Consultar los secrets activos
kubectl get secrets -o wide 
NAME              TYPE                                  DATA   AGE
dashboard-token   kubernetes.io/service-account-token   3      77s

## Consultar la descripción del secret dashboard
kubectl describe secrets dashboard-token 
Name:         dashboard-token
Namespace:    default
Labels:       ["none"]
Annotations:  kubernetes.io/service-account.name: dashboard-sa
              kubernetes.io/service-account.uid: d1aba15d-3dc6-4440-9f24-829ff1326aa4

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     570 bytes
namespace:  7 bytes
token:      Big_Data_Crythed

## Crear un secreto imperativo para una conexión con un servidor SQL:
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
secret/db-secret created

## Crear el pod para usar el secreto:
nano webapp-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
  labels:
    name: webapp-pod
  namespace: default
spec:
  containers:
  - name: webapp
    image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    envFrom:
    - secretRef:
        name: db-secret

## 49.1º Los pod multicontendor ofrece servicio a más de dos contenedores simultáneos a partir de microservicios para distribuir código independiente y reutilizable. Facilitando la escalada del sistema, incluso pueden trabajar juntos en un mismo pod. La función multicontenedor comparte un ciclo de vida dentro del pod, cuando es creado, es eliminado al realizar el comando de borrado, si la aplicación está dentro del pod del servidor también se borrará al estar dentro del mismo pod, lleva el mismo parámetro de red y almacenamiento.

## Muestra de un pod que lleve dos contenedores
nano webapp-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
  namespace: default
spec:
  containers:
  # Container 1
  - name: web-app # array
    image: web-app
    ports:
      - containerPort: 8080
  # Container 2    
  - name: main-app
    image: main-app

## 49.2º Los pods Co-Located Containers arrancan los dos contenedores a la vez, su contenido se expresa con arrays.
nano webapp-pod_Co-Located.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
  namespace: default
spec:
  containers:
  - name: web-app
    image: web-app
    ports:
      - containerPort: 8080
  - name: main-app
    image: main-app

## 49.3º Los pods Regular Init Containers tienen un ciclo de vida determinado, cuando termina un contenedor le sigue el siguiente hasta llegar al contenedor principal y cuando termina el pod se detiene.
nano webapp-pod_Regular_Init_Containers.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
  namespace: default
spec:
  containers:
  - name: web-app
    image: web-app
    ports:
      - containerPort: 8080
  initContainers:
  - name: db-checker
    image: busybox
    command: "wait-for-db-to-start.sh"
  - name: api-checker
    image: busybox
    command: "wait-for-another-api.sh"

## 49.4º Los pods Sidecar Containers tienen una configuración similar a los Regular Init Containers, las diferencias son: el contenedor principal arranca sin esperar a que se acabe los contendores anteriores y se reinician durante el fin de ciclo de vida.
nano webapp-pod_Sidecar_Containers.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
  namespace: default
spec:
  containers:
  - name: web-app
    image: web-app
    ports:
      - containerPort: 8080
  initContainers:
  - name: log-shipper
    image: busybox
    command: "setup-log-shipper.sh"
    restartPolicy: Always

## Filtrar los contenedores del pod por nombre:
kubectl get pods  blue -o yaml | grep name

## Crear un pod con dos contenedores y un comando de ejecución que ponga a un contenedor en reposo:
nano yellow_pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: yellow
  labels:
    name: yellow
  namespace: default
spec:
  containers:
  - name: lemon
    image: busybox
    command: ["sleep"]
    args: ["1000"]
    ports:
      - containerPort: 8080
  - name: gold
    image: redis

## Aplica el pod yellow_pod.yaml:
kubectl apply -f yellow_pod.yaml

## Verificar los registros del namespace elastic-stack y de kibana:
kubectl -n elastic-stack logs kibana 

## Describir el pod app del namespace elastic-stack:
kubectl describe pod app -n elastic-stack

## Revisar el log del namespace elastic-stack:
kubectl -n elastic-stack exec -it app -- cat /log/app.log

## Crear el pod app para supervisar:
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: app
  name: app
  namespace: elastic-stack
spec:
  initContainers:
  - name: sidecar
    image: kodekloud/filebeat-configured
    restartPolicy: Always
    volumeMounts:
      - name: log-volume
        mountPath: /var/log/event-simulator

  containers:
  - image: kodekloud/event-simulator
    name: app
    resources: {}
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - hostPath:
      path: /var/log/webapp
      type: DirectoryOrCreate
    name: log-volume

## Filtrar el initContainer de los pods
kubectl describe pods blue | grep  initContainer
  Normal  Pulling    98s   kubelet            spec.initContainers{init-myservice}: Pulling image "busybox"
  Normal  Pulled     98s   kubelet            spec.initContainers{init-myservice}: Successfully pulled image "busybox" in 562ms (562ms including waiting). Image size: 2236931 bytes.
  Normal  Created    98s   kubelet            spec.initContainers{init-myservice}: Container created
  Normal  Started    98s   kubelet            spec.initContainers{init-myservice}: Container started

## Filtrar la descripción de un pod por estado
kubectl describe pods purple | grep State*
Status:           Pending
    State:          Running
    State:          Waiting
    State:          Waiting

## Filtrar el initContainer de los pods una vez más
kubectl describe pods purple | grep init*
  Normal  Pulled     5m6s  kubelet            spec.initContainers{warm-up-1}: Container image "busybox:1.28" already present on machine and can be accessed by the pod
  Normal  Created    5m6s  kubelet            spec.initContainers{warm-up-1}: Container created
  Normal  Started    5m6s  kubelet            spec.initContainers{warm-up-1}: Container started

## Reemplaza un pod dañado por uno nuevo
nano red_pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: red
  namespace: default
spec:
  containers:
  - command:
    - sh
    - -c
    - echo The app is running! && sleep 3600
    image: busybox:1.28
    name: red-container
  initContainers:
  - image: busybox
    name: red-initcontainer
    command: 
      - "sleep"
      - "20"

## Verifica los eventos de un pod
kubectl describe pods orange | grep -A20 Event

## 50.1º El sistema de autoescalado evalua cuando y como debe escalar los servidores, pods, namespaces y nodos activos. Cuando se ubaba mucho los servidores físicos se escalaban cuando había un margen para ofrecer más volumen de software, cuando se llegaban más usuarios y el servidor no lo afrontaba; se desconecaba, se cambiaba el procesador, memoria, se aumentaba el espacio en disco y se actualizaba las aplicaciones necesarias. Este tipo de escalado es el escalado vertical.

## 50.2º Mientras que el escalado horizontal consiste en agregar un servidor nuevo que ofrece los mismos servicios que los demás servidores activos, evitando apagar el servidor inicial.

## 50.3º El osquestardor que integra kubernetes realiza esta operación. Tiene que elegir entre escalar la infraestructura del cluster y/o scalar la carga de trabajo.

## 50.4º El escalado de cluster puede ser vertical y/o horizontal, en el vertical añade más recursos al contenedor existente, en el horizontal añade más pods.

## 50.5º El escalado de trabajo de carga puede ser vertical y/o horizontal, en el vertical se añade más recursos al contenedor existente para que ofrezca más capacidad, en el horizontal añade contenedores nuevos para complementar el contenedor existente.

## 50.6º Cuando se hace manualmente un escalado de cluster se usa el comando a nivel horizontal:
kubeadm join

## 50.7º Cuando se hace manualmente un escalado de carga de trabajo se usa el comando a nivel horizontal:
kubeadm scale

## 50.8º Para escalar manualmente la carga de trabajo en nivel vertical se una el comando:
kubectl edit

## 50.9º Las opciones para un escalado automático son: ["Cluster_Autoscaler", "Horizontal_Pod_Autoscaler","Vertical_Pod_Autoscaler"]

## 51.1º El escalado horizontal manual se aplicará cuando el pod activo esté alcanzando cerca del umbral de recursos máximos alcanzados.

## Se establece un deployment con limitación:
nano deployment_limit.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment_limit
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "250m"
          limits:
            memory: "500m"

## El comando para monitorizar es:
kubectl top pod nginx-deployment_limit

## Si necesitamos mejorarlo usamos el comando:
kubectl scale deployment nginx-deployment_limit --replicas=3

## 51.2º Cuando está activado el autoscalado horizontal de pods, monitoriza a los pods, añade recursos nuevos cunado los necesita y los borra cuando dejan de ser necesarios.

## El comando para un escalado horizontal de pods automático es:
kubectl autoscale deployment nginx-deployment_limit \
--cpu-percent=50 --min=1 --max=10

## El HPA puede ser supervisado con el comando:
kubectl get hpa -o wide

## Para borrar el autoescalado usaremos:
kubectl delete hpa nginx-deployment_limit

## Crear un deploy que incluya un servicio interno:
nano deployment.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask
        image: rakshithraka/flask-web-app
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: flask-web-app-service
spec:
  type: ClusterIP
  selector:
    app: flask-app
  ports:
   - port: 80
     targetPort: 80  

## Escalar el deploy con un comando:
kubectl scale deployment --replicas=3 flask-web-app 
deployment.apps/flask-web-app scaled

## crear un nuevo deploy para verificar su estado inicial y modificarlo después
nano deployment.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 7
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
#        resources:
#         requests:
#           cpu: 100m
#        limits:
#           cpu: 200m

## Crea un fichero de autoescalado horizontal
nano autoscale.yml 
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  creationTimestamp: null
  name: nginx-deployment
spec:
  maxReplicas: 3
  metrics:
  - resource:
      name: cpu
      target:
        averageUtilization: 80
        type: Utilization
    type: Resource
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
status:
  currentMetrics: null
  desiredReplicas: 0
  currentReplicas: 0

## Aplica el fichero
kubectl apply -f autoscale.yml 
horizontalpodautoscaler.autoscaling/nginx-deployment created

## El fichero de despliegue tiene un bug, corrige el bug y vuelve a desplegarlo, verifica si está bien con este comando:
kubectl get hpa --watch
NAME               REFERENCE                     TARGETS       MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   cpu: 0%/80%   1         3         3          7m28s
nginx-deployment   Deployment/nginx-deployment   cpu: 0%/80%   1         3         1          7m30s

## 52.1º Redimensiar los pods permiten modificarlos y estar operativos, por defecto la mejor forma de actualizar un pod es borrando y crearlo otra vez, esto perjudica a los que están conectados al servicio del pod. Se estima que estará activada por defecto en alguna versión futura del sistema kubernetes. Necesitamos activar la función FEATURE_GATES=InPlacePodVerticalScaling=true

## 52.2º Limitaciones a la hora de escalar:
## 1º Solo la CPU y RAM pueden ser cambiados
## 2º La clase POD y QoS no pueden cambiar
## 3º Los contenedores no pueden ser redimensionados
## 4º Los limitadores de recursos no pueden ser borrados
## 5º La limitación de memoria de un contenedor no puede establecerse por debajo del uso actual; si el contenedor está en este estado, el redimensionador estará mostrará 'En Progreso' hasta que el limite de memoria sea posible
## 6º Las ventanas de los pods no pueden redimensionar

## 52.3º En el escalado vertical necesitamos editar el fichero y aplicar los cambios necesarios

## Crear un deploy que debemos actualizar después:
nano deployment.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        resources:
         requests:
           cpu: 250m
        limits:
           cpu: 500m

## Verificar el despliegue
kubectl top nginx-deployment

## Editamos el fichero, solo tenemos que editar la linea de requests.cpu
kubectl edit deployment nginx-deployment
(Localiza los limitadores y establece)
spec.containers.resources.requests.cpu: 1

## 53.3º El autoescalador vertical de pods evalúa las métricas, añade recursos nuevos y balancea el estado garantizando la estabilidad. 

## Crear un fichero para el autoescalado:
nano vertical-pod-autoscaler.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app"
        minAllowed:
          cpu: "250"
        maxAllowed:
          cpu: "2"
        controlledResources: ["cpu"]


## Obtener los pods del ns kube-system
kubectl get pods -n kube-system | vpa

## 53.4º El parámetro updatePolicy.updateMode se puede configurar con varias opciones:
["OFF","Solo_recomendaciones;_no_realiza_cambios"]
["Initial","Solo_cambia_en_la_creación_del_pod;_después_no"]
["Recreate","solo_si_el_consumo_está_en_el_rango"]
["Auto","actualiza_pods_existentes_para_numeros_recomendados,_por_ahora,es_como_el_modo_recreate_pero_cuando_el_In-place-update_está_soportado_por_pod_resources_esté_disponible_este_es_el_modo_preferido"]

## Describimos el vpa creado
kubectl describe vpa my-app-vpa

## 53.5º Diferencias entre Escalada Vertical y Horizontal
["Scaling_Method"],["Incrementa_CPU_&_RAM"],["Añade/Borra_Pods_basados_en_carga"]
["Pod_Behavior"],["Reduce_Pods_al_aplicar_nuevos_valores"],["Mantiene_los_pods_existentes"]
["Handles_Traffic_Spikes?"],["Requiere_el_reinicio_de_pod"],["Añade_nuevos_pods"]
["Optimizes_Cost?"],["Previene_el_sobre-aprovisionamiento_de_recuros"],["Anula_los_pod_innecesarios"]
["Best_for"],["Trabajos_estables"],["Aplicaiones_web,microservicios"]
["Example_Uses_Cases"],["Bases_de_datos"],["Servicios_web,microservicio"]

## Descargar el repositorio de autoscaler
git clone https://github.com/kubernetes/autoscaler.git

## Ubicarse en --> autoscaler/vertical-pod-autoscaler/hack/
cd autoscaler/vertical-pod-autoscaler/hack/

## Verificar si el fichero vpa-up.sh es ejecutable
chmod -v +x vpa-up.sh 
./vpa-up.sh 

## Verificar los crds y filtrar por verticalpodautoscaler
kubectl get crds | grep verticalpodautoscaler

## Verificar los pods por el ns kube-system y filtrar por vpa
kubectl get pods --namespace kube-system | grep vpa

## Crear el fichero flask-app.yml
nano flask-app.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
  labels:
    app: flask-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image:  kodekloud/flask-session-app:1 
        ports:
        - name: http
          containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: flask-app-service
  labels:
    app: flask-app
spec:
  selector:
    app: flask-app
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
---
apiVersion: "autoscaling.k8s.io/v1"
kind: VerticalPodAutoscaler
metadata:
  name: flask-app
spec:
  # recommenders field can be unset when using the default recommender.
  # When using an alternative recommender, the alternative recommender's name
  # can be specified as the following in a list.
  # recommenders: 
  #   - name: 'alternative'
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: flask-app
  updatePolicy:
    updateMode: "Recreate"
    evictionRequirements:
      - resources: ["cpu", "memory"]
        changeRequirement: TargetHigherThanRequests
  resourcePolicy:
    containerPolicies:
      - containerName: '*'
        minAllowed:
          cpu: 100m
          memory: 100Mi
        maxAllowed:
          cpu: 1
          memory: 500Mi
        controlledResources: ["cpu", "memory"]

## Desplegar el fichero flask-app.yml --> fichero de deploy
kubectl create -f flask-app.yml 

## Verificar los logs del flask-app --> 1
kubectl logs flask-app-8676bb879b-8tc6k 

## Verificar los logs del flask-app --> 2
kubectl logs replicasets/flask-app-8676bb879b 

## Escalar el deploy flask-app a 2 replicas
kubectl scale deployment flask-app --replicas=2

## Evaluar el deploy
kubectl get deployment flask-app -o wide 

## Evaluar los pods de flask-app
kubectl get pods -l app=flask-app

## Describir el vpa de flask-app
kubectl describe vpa flask-app

## Crear el fichero vpa-cpu-test.yml
nano vpa-cpu-testing.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app-4
  labels:
    app: flask-app-4
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-app-4
  template:
    metadata:
      labels:
        app: flask-app-4
    spec:
      containers:
      - name: flask-app-4
        image:  kodekloud/flask-session-app:1 
        ports:
        - name: http
          containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: flask-app-4-service
  labels:
    app: flask-app-4
spec:
  type: NodePort
  selector:
    app: flask-app-4
  ports:
  - name: http
    protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080

## Evaluar el top de los pod
kubectl top pod 
error: metrics not available yet

## Crear el fichero vpa-cpu.yml
nano vpa-cpu.yml 
---
apiVersion: "autoscaling.k8s.io/v1"
kind: VerticalPodAutoscaler
metadata:
  name: flask-app
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: flask-app-4
  updatePolicy:
    updateMode: "Off"  # You can set this to "Auto" if you want automatic updates
  resourcePolicy:
    containerPolicies:
      - containerName: '*'
        minAllowed:
          cpu: 100m
        maxAllowed:
          cpu: 1000m
        controlledResources: ["cpu"]

## Aplicar el fichero yaml
kubectl apply -f vpa-cpu.yml 
verticalpodautoscaler.autoscaling.k8s.io/flask-app created

## Verificar los vpa
kubectl get vpa
NAME        MODE   CPU   MEM   PROVIDED   AGE
flask-app   Off                           5s

## Crear un scrip bash y usar una 2º Ventana de consola
nano load.sh 
#!/bin/bash

echo "Load initiated in the background. Please do not terminate this process."

timeout 1000s bash -c 'for i in {1..10}; do (while true; do curl -s http://controlplane:30080 > /dev/null; done) & done; wait'

## Crear el fichero flask-app_record.yaml
nano flask-app_record.yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  annotations:
  name: flask-app
  namespace: default
spec:
  resourcePolicy:
    containerPolicies:
    - containerName: '*'
      controlledResources:
      - cpu
      maxAllowed:
        cpu: 1000m
      minAllowed:
        cpu: 100m
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: flask-app-4
  updatePolicy:
    updateMode: "Off"
status:
  conditions:
  - lastTransitionTime: "2024-11-07T06:18:17Z"
    status: "True"
    type: RecommendationProvided
  recommendation:
    containerRecommendations:
    - containerName: flask-app-4
      lowerBound:
        cpu: 100m
      target:
        cpu: 143m
      uncappedTarget:
        cpu: 143m
      upperBound:
        cpu: "1"

## Crear un fichero con el contenido del echo
echo "143m" > /root/target

## Aplicar el fichero.yaml
kubectl apply -f flask-app_record.yaml 
verticalpodautoscaler.autoscaling.k8s.io/flask-app configured

## 54.1º Cuando un nodo deja de funcionar y los pods no están respaldados, la aplicación y los clientes están desconectados, cuando transcurre más de cinco minutos, el sistema da el nodo por perdido, si después se inicia empieza desde cero.

## 54.2º Un parámetro que se puede usar en los ficheros yaml es:
tolerationSeconds: 3600

## 54.3º Para administrar los nodos con problemas se usan los comandos:
kubectl drain node-1
kubectl uncordon node-1
kubectl cordon node-1

## Los nodos drenados pierden el pod pero si está establecido por un deploy se recompone con otro nodo.

## Los nodos marcados con el comando cordon están establecidos como no-programable.

## El comando uncordon retira la marca para recibir pod nuevos.

## Obtener los nodos activos:
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   5m28s   v1.35.0   10.244.170.133   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    ["none"]          4m56s   v1.35.0   10.244.147.128   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Obtener los deploys activos:
kubectl get deployments -o wide 
NAME   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
blue   3/3     3            3           65s   nginx        nginx:alpine   app=blue

## Obtener los pods activos:
kubectl get pods -o wide 
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6466bf85df-d6wf7   1/1     Running   0          2m12s   172.17.1.3   node01         ["none"]           ["none"]
blue-6466bf85df-k57ll   1/1     Running   0          2m12s   172.17.0.4   controlplane   ["none"]           ["none"]
blue-6466bf85df-t4t5q   1/1     Running   0          2m12s   172.17.1.2   node01         ["none"]           ["none"]

## Establece el modo mantenimiento al nodo1 y acordonalo:
kubectl drain node01 --ignore-daemonsets ; kubectl cordon node01
node/node01 already cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-bgxqg, kube-system/kube-proxy-54j7k
evicting pod default/blue-6466bf85df-t4t5q
evicting pod default/blue-6466bf85df-d6wf7
pod/blue-6466bf85df-t4t5q evicted
pod/blue-6466bf85df-d6wf7 evicted
node/node01 drained
node/node01 already cordoned

## El nodo01 se marcará como desactivado después de detenerlo:
kubectl get nodes -o wide 
NAME           STATUS                     ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready                      control-plane   13m   v1.35.0   10.244.170.133   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready,SchedulingDisabled   ["none"]          12m   v1.35.0   10.244.147.128   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Después del mantenimiento vuelve a activarlo:
kubectl uncordon node01 
node/node01 uncordoned

## El nodo no recibió ningún pod existenete:
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   15m   v1.35.0   10.244.170.133   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    ["none"]          14m   v1.35.0   10.244.147.128   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
##
kubectl get pods -o wide 
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6466bf85df-bbvcc   1/1     Running   0          3m27s   172.17.0.6   controlplane   ["none"]           ["none"]
blue-6466bf85df-frfnn   1/1     Running   0          3m27s   172.17.0.5   controlplane   ["none"]           ["none"]
blue-6466bf85df-k57ll   1/1     Running   0          9m38s   172.17.0.4   controlplane   ["none"]           ["none"]

## Describe el nodo controlplane:
kubectl describe nodes controlplane

## El nodo node01 necesita otro mantenimiento:
node/node01 cordoned
error: unable to drain node "node01" due to error: cannot delete cannot delete Pods that declare no controller (use --force to override): default/hr-app, continuing command...
There are pending nodes to be drained:
 node01
cannot delete cannot delete Pods that declare no controller (use --force to override): default/hr-app
node/node01 already cordoned

## El pod hr-app está bloqueando la desconexión del pod:
kubectl get pods -o wide 
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6466bf85df-bbvcc   1/1     Running   0          8m42s   172.17.0.6   controlplane   ["none"]           ["none"]
blue-6466bf85df-frfnn   1/1     Running   0          8m42s   172.17.0.5   controlplane   ["none"]           ["none"]
blue-6466bf85df-k57ll   1/1     Running   0          14m     172.17.0.4   controlplane   ["none"]           ["none"]
hr-app                  1/1     Running   0          2m19s   172.17.1.4   node01         ["none"]           ["none"]

## Acordona el nodo node01
kubectl cordon node01
node/node01 cordoned

## Verifica el estado de los pods:
kubectl get pods -o wide 
NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6466bf85df-bbvcc     1/1     Running   0          14m     172.17.0.6   controlplane   ["none"]           ["none"]
blue-6466bf85df-frfnn     1/1     Running   0          14m     172.17.0.5   controlplane   ["none"]           ["none"]
blue-6466bf85df-k57ll     1/1     Running   0          20m     172.17.0.4   controlplane   ["none"]           ["none"]
hr-app-759cb554bf-sqt2z   1/1     Running   0          4m10s   172.17.1.5   node01         ["none"]           ["none"]

## 55º Las versiones de los componentes de kubernetes son compatibles entre sí sin necesidad de tener el más reciente pero si necesita actualizar se puede realizar desde los comandos: apt update ; apt full-upgrade && kubeadm upgrate. A parte de elegir las versiones de los componentes, cluster, nodos, etc.

## Verificar la versión de los nodos:
kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   25m   v1.34.0   10.244.127.208   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26
node01         Ready    ["none"]          24m   v1.34.0   10.244.56.39     ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26

## Localiza los taints en los dos nodos a través de la descripción:
kubectl describe nodes node01 | grep -i taint
Taints:             ["none"]

kubectl describe nodes controlplane | grep -i taint
Taints:             ["none"]

## Localiza los deploy activos
kubectl get deployments -o wide 
NAME   READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
blue   5/5     5            5           3m59s   nginx        nginx:alpine   app=blue

## Verifica la version de kubeadm
kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"34", EmulationMajor:"", EmulationMinor:"", MinCompatibilityMajor:"", MinCompatibilityMinor:"", GitVersion:"v1.34.0", GitCommit:"f28b4c9efbca5c5c0af716d9f2d5702667ee8a45", GitTreeState:"clean", BuildDate:"2025-08-27T10:15:59Z", GoVersion:"go1.24.6", Compiler:"gc", Platform:"linux/amd64"}

## Localizar que versiones de kubeadm se pueden instalar:
kubeadm upgrade plan
kubeadm upgrade plan | grep v

## Para actualizar la versión del nodo ["controlplane"] debemos detenerlo:
kubectl drain controlplane --ignore-daemonsets ; kubectl drain node01 --ignore-daemonsets
node/controlplane cordoned
Warning: ignoring DaemonSet-managed Pods: kube-flannel/kube-flannel-ds-mlqhr, kube-system/kube-proxy-rm9tp
evicting pod kube-system/coredns-6678bcd974-stgmp
evicting pod default/blue-759779556-clrhz
evicting pod kube-system/coredns-6678bcd974-89zdg
evicting pod default/blue-759779556-gtg7q
pod/blue-759779556-gtg7q evicted
pod/blue-759779556-clrhz evicted
pod/coredns-6678bcd974-89zdg evicted
pod/coredns-6678bcd974-stgmp evicted
node/controlplane drained

## Agregamos en la ruta ["/etc/apt/source.list.d/kubernetes.list"] el código de la fuente de actualización:
# deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /

## Actualizamos el sistema completo:
apt update ; apt full-upgrade ; apt-cache madison kubeadm

## Instalamos la nueva versión de kubeadm:
apt-get install kubeadm=1.35.0-1.1

## Actualizamos el cluster de kubernetes. Si pregunta sobre si quieres continuar, escribe 'Y+Enter_key':
kubeadm upgrade plan v1.35.0 ; kubeadm upgrade apply v1.35.0

## Reinicia los procesos y servicios del kubelet:
systemctl daemon-reload ; systemctl restart kubelet

## Estado de los nodos después de actualizar:
kubectl get nodes -o wide 
NAME           STATUS                     ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready,SchedulingDisabled   control-plane   60m   v1.35.6   10.244.127.208   ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26
node01         Ready,SchedulingDisabled   ["none"]          60m   v1.34.0   10.244.56.39     ["none"]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26

## Reactivar los nodos en reposo:
kubectl uncordon controlplane ; kubectl uncordon node01 
node/controlplane uncordoned
node/node01 uncordoned

## Volvemos a realizar otro mantenimiento a los nodos:
kubectl drain controlplane --ignore-daemonsets ; kubectl drain node01 --ignore-daemonsets

## ssh node01 y dentro repetir los procesos anteriores

## 56º Los elementos creados en kubernetes se pueden respaldar para ser restaurados a petición de administrador de sistemas. El sistema guarda un una base de datos con los comandos:

## Obtener todos los servicios activos y exportarlos al fichero de respaldo:
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml

## Consultar el etcd.service:
etcd.service

## Crear una copia de seguridad con etcdctl:
etcdctl snapshot save </path/to-file.db> --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key
## listar el directorio:
ls -lh

## Detemos el servicio apiserver de kubernetes:
systemctl stop kube-apiserver

## Restaurar una instantanea del servicio respaldado. Cuando se usa este comando crea un directorio nuevo para no se solape con otro cluster:
etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir=/var/lib/etcd-from-backup

## Consultar el etcd.service de nuevo:
etcd.service

## Reiniciamos los procesos y el apiserver vuelve a trabajar:
systemctl daemon-reload ; systemctl restart etcd ; systemctl start kube-apiserver

## Verificar versión del comando:
etcdctl version

## Verificar los deploy activos:
kubectl get deployments -o wide 
NAME   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
blue   3/3     3            3           2m    nginx        nginx:alpine   app=blue
red    2/2     2            2           2m    nginx        nginx:alpine   app=red

## Verificar el etcdctl:
etcdctl version
etcdctl version: 3.5.16
API version: 3.5

## Filtrar la version de imagen de la descripción del pod etcd-controlplane en el ns kube-system
kubectl describe pod etcd-controlplane  -n kube-system | grep Image:
    Image:         registry.k8s.io/etcd:3.6.6-0

## Filtrar las URL https de la descripción del pod etcd-controlplane en el ns kube-system y verificar la linea --listen-client-urls
kubectl describe pod etcd-controlplane  -n kube-system | grep https://
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.244.253.186:2379
      --advertise-client-urls=https://10.244.253.186:2379
      --initial-advertise-peer-urls=https://10.244.253.186:2380
      --initial-cluster=controlplane=https://10.244.253.186:2380
      --listen-client-urls=https://127.0.0.1:2379,https://10.244.253.186:2379 # --> LOOK AT THIS
      --listen-peer-urls=https://10.244.253.186:2380

## Filtrar las rutas del servidor de certificados de la descripción del pod etcd-controlplane en el ns kube-system :
kubectl describe pod etcd-controlplane  -n kube-system | grep /etc/kubernetes
      --cert-file=/etc/kubernetes/pki/etcd/server.crt --> LOOK AT THIS
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
    Path:          /etc/kubernetes/pki/etcd

## Filtrar las rutas del CA de certificados de la descripción del pod etcd-controlplane en el ns kube-system :
kubectl describe pod etcd-controlplane  -n kube-system | grep /etc/kubernetes
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --> LOOK AT THIS
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
    Path:          /etc/kubernetes/pki/etcd

## Crea una copia de seguridad para el sistema:
etcdctl snapshot save /opt/snapshot-pre-boot.db --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key
{"level":"info","ts":"2026-07-12T17:41:05.548222Z","caller":"snapshot/v3_snapshot.go:65","msg":"created temporary db file","path":"/opt/snapshot-pre-boot.db.part"}
{"level":"info","ts":"2026-07-12T17:41:05.552023Z","logger":"client","caller":"v3@v3.5.16/maintenance.go:212","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":"2026-07-12T17:41:05.552050Z","caller":"snapshot/v3_snapshot.go:73","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":"2026-07-12T17:41:05.558340Z","logger":"client","caller":"v3@v3.5.16/maintenance.go:220","msg":"completed snapshot read; closing"}
{"level":"info","ts":"2026-07-12T17:41:05.558397Z","caller":"snapshot/v3_snapshot.go:88","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"1.8 MB","took":"now"}
{"level":"info","ts":"2026-07-12T17:41:05.558450Z","caller":"snapshot/v3_snapshot.go:97","msg":"saved","path":"/opt/snapshot-pre-boot.db"}
Snapshot saved at /opt/snapshot-pre-boot.db

## Para restaurar y Verificar necesitamos usar etcdutl:
etcdutl --write-out=table snapshot status snapshot.db
+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| fe01cf57 |       10 |          7 | 2.1 MB     |
+----------+----------+------------+------------+

## Mover el fichero kube-apiserver.yaml a directorio /tmp/ y esperar 30 seg:
mv -v /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/ ; sleep 30
copied '/etc/kubernetes/manifests/kube-apiserver.yaml' -> '/tmp/kube-apiserver.yaml'
removed '/etc/kubernetes/manifests/kube-apiserver.yaml'

## Restaurar el sistema con el fichero de copia /opt/backup_file.db:
etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir=/var/lib/etcd-from-backup
2026-07-12T17:56:23Z    info    snapshot/v3_snapshot.go:265     restoring snapshot      {"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-from-backup/member/wal", "data-dir": "/var/lib/etcd-from-backup", "snap-dir": "/var/lib/etcd-from-backup/member/snap", "initial-memory-map-size": 10737418240}
2026-07-12T17:56:23Z    info    membership/store.go:141 Trimming membership information from the backend...
2026-07-12T17:56:23Z    info    membership/cluster.go:421       added member    {"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"]}
2026-07-12T17:56:23Z    info    snapshot/v3_snapshot.go:293     restored snapshot       {"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-from-backup/member/wal", "data-dir": "/var/lib/etcd-from-backup", "snap-dir": "/var/lib/etcd-from-backup/member/snap", "initial-memory-map-size": 10737418240}

## Editamos el fichero movido. La parte inferior del fichero se le cambia el path:
nano /etc/kubernetes/manifests/etcd.yaml
  volumes:
  - hostPath:
      path: /etc/kubernetes/pki/etcd
      type: DirectoryOrCreate
    name: etcd-certs
  - hostPath:
      path: /var/lib/etcd-from-backup        # NEW restored directory
      type: DirectoryOrCreate
    name: etcd-data

## El fichero debe ser devuelto a su sitio:
mv -v /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/
copied '/tmp/kube-apiserver.yaml' -> '/etc/kubernetes/manifests/kube-apiserver.yaml'
removed '/tmp/kube-apiserver.yaml'

## Reiniciamos el kube-controller-manager, movemos el kube-controller.yaml a /tmp/, esperamos 20 seg, devolverlo a su sitio y repetir con kube-scheduler:
mv -v /etc/kubernetes/manifests/kube-controller-manager.yaml /tmp/ ; sleep 20 ; mv -v /tmp/kube-controller-manager.yaml /etc/kubernetes/manifests/ ; mv -v /etc/kubernetes/manifests/kube-scheduler.yaml /tmp/ ; sleep 20 ; mv -v /tmp/kube-scheduler.yaml /etc/kubernetes/manifests/ ; systemctl restart kubelet

## Verificar los procesos críticos:
watch crictl ps

## Verificar todos los recursos de los namespaces:
kubectl get deployments,services --all-namespaces

## Verifica los pods con todos los ns y los nodos
kubectl get pods --all-namespaces
kubectl get nodes -o wide
