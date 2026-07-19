## 31º La afinidad garantiza que los pods se alojen en el nodo concreto que vayamos a elegir, los pods que llevan carga ligera se introducen en los nodos que soporten este tipo de carga.

## Muestra de fichero yaml con selector de nodo Largo:
nano pod-definition_1.yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  nodeSelector:
    size: Large

## Muestra de fichero yaml con la afinidad
nano pod-definition_2.yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
     nodeSelectorTerms:
     - matchExpressions:
       - key: Size
         operator: In / NotIn / Exists
         values:
         - Large
         - Medium # --> este valor es opcional
##       - Small
## Cuando se establece el operador Exists no se aplica el filtro de tamaños
## El parámetro con el tamaño de una frase sin espacios es para garantizar la integridad en caso de cambiar el nombre del nodo

## Tipos de afinidad de nodo:
## los disponibles --> requiredDuringSchedulingIgnoredDuringExecution y preferredDuringSchedulingIgnoredDuringExecution
## Los planeados --> requiredDuringSchedulingRequiredDuringExecution

## DuringScheduling: es el estado en el que el pod no está creado y se creará
## DuringExecution: es el estado en el que el pod ha estado en ejecución activa

## Si se quiere crear el pod y el nodo compatible está ausente, se verifica el estado de requerido a través del parámetro Required, al no encontrarse el pod no inicia.
## Cuando necesitamos que esté activo sin importar el nodo requerido, establecemos Preferred y lo coloca en el nodo que esté disponible.
## Si está activo el pod puede modificar la afinidad del nodo, puede suceder con el cambio de nombre del nodo.
## El cambio de nombre o el borrado de la etiqueta del tamaño del nodo puede solucionarse con el parámetro Ignored

## Localizar los label de un nodo:
kubectl describe nodes node01

## Cambiar el label de un nodo:
kubectl label nodes node01 color=blue
node/node01 labeled

## Fichero yaml con despliegue
nano deployment_blue.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue
  labels:
    app: nginx
spec:
  replicas: 3
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

## aplicar fichero yaml
kubectl apply -f deployment_blue.yaml 
deployment.apps/blue created

## Verificación del despliegue de Blue
kubectl get deployments.apps -o wide 
NAME   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
blue   3/3     3            3           81s   nginx        nginx    app=nginx

## Estado de nodos
kubectl get nodes -o wide 
|NAME          | STATUS  | ROLES         |  AGE  | VERSION  | INTERNAL-IP     | EXTERNAL-IP |    OS-IMAGE          | KERNEL-VERSION    | CONTAINER-RUNTIME     |
|--------------|---------|---------------|-------|----------|-----------------|-------------|----------------------|-------------------|-----------------------|
|controlplane  | Ready   | control-plane |  17m  | v1.35.0  | 10.244.56.22    |   ["none"]  |   Ubuntu 22.04.5 LTS |  6.8.0-90-generic |  containerd://1.7.22  |
|node01        | Ready   | ["none"]      |    17m| v1.35.0  | 10.244.34.242   |   ["none"]  |   Ubuntu 22.04.5 LTS |  6.8.0-90-generic |   containerd://1.7.22 |

## Estado de pods
kubectl get pods -o wide 
NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
blue-56c45fd5ff-26hfq   1/1     Running   0          2m3s   172.17.1.2   node01         ["none"]           ["none"]
blue-56c45fd5ff-8b67j   1/1     Running   0          2m3s   172.17.1.3   node01         ["none"]           ["none"]
blue-56c45fd5ff-8zhnl   1/1     Running   0          2m3s   172.17.0.4   controlplane   ["none"]           ["none"]

## Creación del fichero yaml del despliegue de Red
nano deployment_red.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
spec:
  replicas: 2
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/control-plane
                operator: Exists

## 32º Cuando establecemos la clasificación desde el tain o la afinidad también es posible que un pod que no debe colocarse en el nodo no correspondiente, un pod con la tolerancia a un tain puede entrar en un nodo que no lleva tain. Con el uso de afinidad establecemos una etiqueta a los nodos, después establecenos los selectores a los pods que necesitamos ubicar en los nodos con los identificadores, incluso esta situación no impide que otros pods se coloquen en el nodo no compatible con la operación de despliegue. Una solución es combinar las dos opciones: 1º Se establece los tain & tolerations para clasificar. 2º Aplicamos etiquetas para clasificar y filtrar con el node affinity

## 33º Cada nodo tiene un límite de recursos *'CPU & RAM', el planificador kube-Scheduler va verificando donde se puede añadir hasta completar la carga de sistema, si un nodo está lleno y no admite más pods, se lleva a otro disponible. Cuando todos están llenos el siguiente pod estará en estado pendiente de colocar en el nodo. Normalmente los pods llevan 1 Nucleo de CPU y 1 GB de RAM, estos valores son configurables a través de los ficheros yaml.

nano pod-definition_1.yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  resources:
    requests:
      memory: "4Gi" / "4G"
      cpu: 2

## A parte del Procesador y Memoria Max, se puede elegir el límite de seguirad de los recursos.

nano pod-definition_1.yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-processor
  resources:
    requests:
      memory: "1Gi" / "1G" # --> recursos estándares del pod
      cpu: 2
    limits:
      memory: "2Gi" / "2G" # --> límite máximo del pod
      cpu: 2

## El sistema se asegura que no se vaya a superar el límite para proteger el hardawre, kernel y peticiones. Si los llega a superar, el sistema le retira los recursos y se detiene el pod, después se genera un registro con el mensaje de error por parada inesperada

## En los valores predefinidos, el pod puede usar todo lo que necesite, incluso es capaz de estrangular/asfixiar el Procesador & Memoria si no se controla el acceso a los recursos y afectando a otros pods del nodo
|                                      |            |
|                                      | BEST_OPTION|
NO REQUESTS  | NO REQUESTS |  REQUESTS |    REQUESTS|
NO LIMITS    |    LIMITS   |  LIMITS   | NO LIMITS
|------------|-------------|-----------|------------|
 ##  | #     |     |       |     |     | #    | #
 #   | #     |     |       |     |     | #    | #
 #   | #     |3vCPU| 3Gi   |3vCPU| 3Gi | #    | #
 #   | #     | # # | # #   | #   | #   | # #  | # #
 # # | # #   | # # | # #   |1vCPU| 1Gi |1vCPU | 1Gi
| 1 2 | 1 2   | 1 2 | 1 2   | 1 2 | 1 2 | 1 2  | 1 2 
| CPU | RAM   | CPU | RAM   | CPU | RAM | CPU  | RAM
|             REQUESTS=LIMITS                       |
## Los pods cuando compiten entre sí y no se raciona bien el acceso es capaz de detener a otros pods por falta de recursos. Cuando es la memoria el recurso al borde de colapsar, el pod que gaste más debe ser detenido para liberar los recursos de RAM.

## fichero yaml para limitar el rango de cpu:
nano limit-range-cpu.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m # --> Limit
    defaultRequest:
      cpu: 500m # --> Request
    max:
      cpu: "i" # --> Limit
    min:
      cpu: 100m # --> Request
    type: Container

## fichero yaml para limitar el rango de memoria:
nano limit-range-memory.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint
spec:
  limits:
  - default:
      memory: 1Gi # --> Limit
    defaultRequest:
      memory: 1Gi # --> Request
    max:
      memory: 1Gi # --> Limit
    min:
      memory: 600Mi # --> Request
    type: Container
  
## Si fuese necesario racionarlo todo a nivel de nodos o de cluster, se puede establecer unas cuotas a nivel de los namespaces a través de límites duros:
nano resources-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi

## 34.1º A la hora de modificar los pods, algunos valores no deben ser modificados, lo que esté debajo del spec, como las variables de entorno, cuentas de servicio y limitador de recursos cuando el pod está activado. Los cambios fallidos son restaurados a través de un fichero temporal que cuando el cambio es correcto se sobreescribe, los valores de las variables de entorno no son editables.

## 34.2º La mejor opción es obtener los valores del pod y redigirlo a un fichero yaml:
kubectl get pod webapp -o yaml > my-new-pod.yaml

## 34.3º Editar el fichero exportado:
nano my-new-pod.yaml

## 34.4º Borrar el pod que no fue editado:
kubectl delete pod webapp

## 34.5º Aplicar el nuevo pod:
kubectl create -f my-new-pod.yaml
kubectl apply  -f my-new-pod.yaml

## 34.6º Para los deployments es más fácil desde el modo edición, la plantilla borra los pod y los vuelve a crear:
kubectl edit deployment my-deployment

## 35º Comparación entre Deployment, ReplicaSet y DaemonSet: En los deployment y replicaset se realizan copias en los diferentes nodos de trabajo, mientras el daemonset son conjuntos de réplicas con los que llamar múltiples estancias.

                      MONITORING      LOGS VIEWER
                  ________|_______________|_______
                 |        |       |       |       |
                 |        |       |       |       |
## DaemonSets    #    |   #   |   #   |   #   |   #   |   #
## ReplicaSets  # #   |  # #  |  # #  |  # #  |  # #  |
## Deployments  # #   |  # #  |  # #  |  # #  |  # #  |
## ----------> NODE1  | NODE2 | NODE3 | NODE4 | NODE5 | NODE6

## Cuando se añade o se borra los pods la información queda en el pod y al borrar se pierde lo que llevaba, los daemonsets aseguran la copia de cada pod en todos los clusters. El proceso administrativo de los pods está al cargo del daemonset sin intervención necesaria. En los delpyment y replicaset el sistema de red estaba a cargo del kube-proxy, en los daemonset lo raeliza el calico-network. 

## El diseño del replicaset y el daemnoset es similar:
nano replicaset-definition.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: monitoring-daemon
spec:
 selectors:
    matchLabels:
       app: monitoring-agent
 template:
   metadata:
     labels:
       app: monitoring-agent 
   spec:
     containers:
     - name: monitoring-agent
       image: monitoring-agent

nano daemonset-definition.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
 name: monitoring-daemon
spec:
 selectors:
    matchLabels:
       app: monitoring-agent
 template:
   metadata:
     labels:
       app: monitoring-agent 
   spec:
     containers:
     - name: monitoring-agent
       image: monitoring-agent

## Ambos ficheros se crean con kubectl create -f ["name_file.yam"]
kubectl create -f replicaset-definition.yaml
kubectl create -f daemonset-definition.yaml

## Para consultar el daemonset usaremos:
kubectl get daemonsets

## Para verificar la estructura usaremos:
kubectl describe daemonset ["daemonset_name"]

## Mostrar todos los daemos activos
kubectl get daemonsets.apps --all-namespaces 
NAMESPACE      NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   kube-flannel-ds   1         1         1       1            1           ["none"]                   12m
kube-system    kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   12m

## Obtener todos los pods de todos los namespaces
kubectl get pods --all-namespaces 

## Obener los pods del namespace determinado
kubectl get pods --namespace kube-flannel

## Obtener la descripción del pod kube-flannel-ds
kubectl describe pod kube-flannel-ds-qqbhv --namespace kube-flannel

## Crear una plantilla de un deployment con yaml editable
kubectl create deployment elasticsearch --image=registry.k8s.io/fluentd-elasticsearch:1.20 -n kube-system --dry-run=client -o yaml > fluentd.yaml

## Convierte el deployment en un daemonset
nano fluentd.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: elasticsearch
  name: elasticsearch
  namespace: kube-system
spec:
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - image: registry.k8s.io/fluentd-elasticsearch:1.20
        name: fluentd-elasticsearch
        resources: {}

## Applica los cambios para crear el daemonset
kubectl create -f fluentd.yaml 
daemonset.apps/elasticsearch created

## 36º Los componentes de la arquitectura de kubernetes administran el acceso de los recursos, si están fuera de servicio el sistema maestro, el control-manager, el etcd_cluster y/o el kube-aspiserver, necesitamos a usar solo al kublet. Puede crear el pod pero no ofrecer los detalles, los pods deben estar dentro de un fichero.yaml preconfigurado para usar la nuestra estructura sin el sistema maestro disponible. Cada fichero debe estar almacenado en --> /etc/kubernetes/manifest. Periodicamente se verifica si hay ficheros para crear los pods asegurando que mantiene el servicio activo si se bloquea la aplicación. Cuando necesita reiniciar lo realiza, tras borrar el fichero del directorio el pod desaparece. Esta forma de crear pods se denomina Static Pod; durante este modo no puede administrar las replicas, despliegues o servicios automáticos, la ruta se puede configurar y establecer al kubelet mientras está activo. Cuando se hace algún cambio al fichero, actualiza el pod para recibir los cambios.

## Configurar la ruta de los pods estáticos:
nano kubeconfig.yaml
staticPodPatch: /etc/kubernetes/manifest

## El fichero kubelet.service debe llevar una opción compatible:
nano kubelet.service
--config=kubeconfig.yaml

## Los pods estáticos se consultan como un contenedor de docker:
docker ps

## Si se usa el comando kubectl get pods aparece con estado ContainerCreating.
kubectl get pods
NAME                READY   STATUS            RESTARTS  AGE
static-web-node01   0/1     ContainerCreating 0         29s

## Usos del modo estático; si se proporciona los ficheros locales al kubelet.service, se ahorra la necesidad de descargar los ficheros binarios en cada despliegue de pod estático. El servicio reinicia los pods con problemas de ejecución.

## Diferencias entre los Pods y DaemonSets
# Static Pods: Creado por el kubelet. Despliega El componente de control de despliegues como Pod Estático. Ignorado por el programador de Kube.
# DaemonSets: Creador por el Controlador de DaemonSet y kube-apiserver. Despliega el Agente de Monitorización, Registro de eventos y Agentes en nodos. Ignorado por el programador de Kube.

## Obtener todos los pods de cualquier namespace
kubectl get pods --all-namespaces

## Obener todos los deployments de cualquier namespace
kubectl get deployments --all-namespaces

## Obtener más información sobre los pods
kubectl get pods --all-namespaces -o wide

## Buscar en el directorio /etc/kubernetes/manifest los ficheros con nombre "kube-api*" y ejecturar 'ls -lh'
find /etc/kubernetes/manifests -type f -name "kube-api*" -exec ls -l {} \;
-rw------- 1 root root 3965 Jul  7 14:02 /etc/kubernetes/manifests/kube-apiserver.yaml

## Después un grep y bucar registry para localizar el elemento con los datos de imagen
find /etc/kubernetes/manifests -type f -name "kube-api*" -exec grep registry {} \;
    image: registry.k8s.io/kube-apiserver:v1.35.0

## Iniciamos un pod sin reinio, imagen_busybox, llamado static-busybox, mostrar el yaml, con el comando sleep 1000 y exportar sus datos a un fichero.yaml
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml

## Verificamos el fichero sin editarlo
nano static-busybox.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: static-busybox
  name: static-busybox
spec:
  containers:
  - command:
    - sleep
    - "1000"
    image: busybox
    name: static-busybox
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

## Aplicar el /etc/kubernetes/manifests/static-busybox.yaml
kubectl apply -f static-busybox.yaml 
pod/static-busybox created

## Editamos el atributo de imagen para añadir otra versión de busybox --> busybox:1.28.4
image: busybox:1.28.4

## Volvemos a aplicarlo
kubectl apply -f static-busybox.yaml 
pod/static-busybox created

## Buscamos el static-greenbox con el buscador de pods
kubectl get pods -o wide | grep static-greenbox
static-greenbox-node01        1/1     Running   0              15s    172.17.1.3   node01         ["none"]           ["none"]

## Entramos en el nodo01 con el ssh
ssh node01
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-124-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.

## Buscamos el fichero config.yaml en find /var/lib/kubelet/
find /var/lib/kubelet/config.yaml
/var/lib/kubelet/config.yaml

## Vista previa del fichero config.yaml
find /var/lib/kubelet/config.yaml -exec cat {} +
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 172.20.0.10
clusterDomain: cluster.local
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
cpuManagerReconcilePeriod: 0s
crashLoopBackOff: {}
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMaximumGCAge: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
    text:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/just-to-mess-with-you
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

## Para facilitar la búsqueda; usamos el grep para localizar la clave staticPodPath
find /var/lib/kubelet/config.yaml -exec grep staticPodPath {} +
staticPodPath: /etc/just-to-mess-with-you

## El valor es un directorio, usamos el comando 'cd' para visitarlo y mostrar el comando ls mostrar contenido
cd /etc/just-to-mess-with-you
ls
greenbox.yaml

## Usa el comando rm -v para borar el fichero.yaml
rm -v greenbox.yaml 
removed 'greenbox.yaml'

## Al ejecutar este comando y filtrar por static-greenbox la consulta debe estar vacía
kubectl get pods --all-namespaces -o wide  | grep static-greenbox
(EMPTY)

## 37º Clases prioritaras. Las prioridades ordenan el acceso de los componentes, bases de datos, aplicaciones críticas, trabajos, es importante asegurar que las cargas de trabajo de mayor prioridad se programen sin interrupción frente a las de baja prioridad, cuando un pod de mayor prioridad no consigue iniciarse, detiene un pod de baja prioridad. Las clases prioritarias no tienen namespaces y no se crean dentro del namespace. Las prioridades se establecen con un número negativo y uno positivo, desde 1.000 millones hasta -2.000 millones. Si el valor positivo supera los 2.000 millones está solapándose con las prioridades del sistema y con los componentes del plano de control.

## 37.2º Comando para obtener las clases de prioridad
kubectl get priorityclass

## 37.3º Creamos ficheros con PriorityClass y Pod para verificar los efectos de cambiar prioridad:
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
 value: 1000000000
 description: "Priority class for mission critical pods"

#--------------------------------------------------
apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - name: web
    image: nginx
    ports:
      - containerPort: 8080
  priorityClassName: high-priority

#-----------------------------------------
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000000
description: "Priority class for mission critical pods"
preemtionPolicy: PreemptLowerPriority # --> Los recursos de baja prioridad pierden el acceso a los recursos
preemtionPolicy: Never # --> Los recursos no serán ofrecidos a otro objeto y se establece una cola de programación, incluso un objeto de baja prioridad puede tener el acceso primero

## Obtener las prioridades del priorityclasses.scheduling.k8s.io 
kubectl get priorityclasses.scheduling.k8s.io 
NAME                      VALUE        GLOBAL-DEFAULT   AGE   PREEMPTIONPOLICY
system-cluster-critical   2000000000   false            16m   PreemptLowerPriority
system-node-critical      2000001000   false            16m   PreemptLowerPriority

## Obtener las prioridades del prioritylevelconfigurations.flowcontrol.apiserver.k8s.io
kubectl get prioritylevelconfigurations.flowcontrol.apiserver.k8s.io 
|NAME            |  TYPE     |  NOMINALCONCURRENCYSHARES |  QUEUES      | HANDSIZE      | QUEUELENGTHLIMIT |  AGE |
|----------------|-----------|---------------------------|--------------|---------------|------------------|------|
|catch-all       |  Limited  |             5             |   ["none"]   |  ["none"]     |     ["none"]     |  17m |   
|exempt          |  Exempt   |             ["none"]      |   ["none"]   |  ["none"]     |     ["none"]     |  17m |     
|global-default  |  Limited  |             20            |     128      |     6         |       50         |  17m |
|leader-election |  Limited  |             10            |     16       |     4         |       50         |  17m |
|node-high       |  Limited  |             40            |     64       |     6         |       50         |  17m |
|system          |  Limited  |             30            |     64       |     6         |       50         |  17m |
|workload-high   |  Limited  |             40            |     128      |     6         |       50         |  17m |
|workload-low    |  Limited  |            100            |     128      |     6         |       50         |  17m |

## Obtener el yaml de la prioridad de system-node-critical
kubectl get priorityclasses system-node-critical -o yaml
apiVersion: scheduling.k8s.io/v1
description: Used for system critical pods that must not be moved from their current
  node.
kind: PriorityClass
metadata:
  creationTimestamp: "2026-07-07T15:34:31Z"
  generation: 1
  name: system-node-critical
  resourceVersion: "67"
  uid: 3675df42-44fc-4727-8004-fdfb233f040d
preemptionPolicy: PreemptLowerPriority
value: 2000001000

## Crear el fichero high-priority.yaml
nano high-priority.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "This priority class is used for high-priority pods."
preemptionPolicy: PreemptLowerPriority 

## Importar el 1º fichero recién creado
kubectl create -f high-priority.yaml 
priorityclass.scheduling.k8s.io/high-priority created

## Crear el fichero low-priority.yaml
nano low-priority.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
globalDefault: false
description: "This priority class is used for low-priority pods."
preemptionPolicy: PreemptLowerPriority

## Importar el 2º fichero recién creado
kubectl create -f low-priority.yaml 
priorityclass.scheduling.k8s.io/low-priority created

## Crear un pod y exportar el yaml. (Este pod se iniciará automáticamente)
kubectl run low-priority --image=ngix -o yaml > low-prio-pod.yaml

## Borramos el pod iniciado
kubectl delete pods low-priority 
pod "low-priority" deleted from default namespace

## Sustituimos el fichero largo por uno más compacto. (Los atributos ausentes se rellena solo)
nano low-prio-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-prio-pod
spec:
  containers:
  - name: nginx
    image: nginx
  priorityClassName: low-priority

## Importarmos el low-prio-pod.yaml
kubectl apply -f low-prio-pod.yaml
pod/low-prio-pod created

## Copiamos el fichero para crearlo con otro nombre
cp -v low-prio-pod.yaml high-prio-pod.yaml 
'low-prio-pod.yaml' -> 'high-prio-pod.yaml'

## Editamos el high-prio-pod.yaml para establecer los nuevos valores
nano high-prio-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-prio-pod
spec:
  containers:
  - name: nginx
    image: nginx
  priorityClassName: high-priority

## Importamos el fichero.yaml
kubectl apply -f high-prio-pod.yaml 
pod/high-prio-pod created

## Comparación de los sistemas de prioridades
kubectl get pods -o custom-columns="NAME:.metadata.name,PRIORITY:.spec.priorityClassName"
NAME            PRIORITY
high-prio-pod   high-priority
low-prio-pod    low-priority

## Consulta de los pods
kubectl get pods
NAME            READY   STATUS    RESTARTS   AGE
critical-app    0/1     Pending   0          25s
high-prio-pod   1/1     Running   0          69s
low-app         1/1     Running   0          25s
low-prio-pod    1/1     Running   0          4m3s

## Consulta de descripción del pod critical-app
kubectl describe pod critical-app
Name:             critical-app
Namespace:        default
Priority:         0
Service Account:  default
Node:             ["none"]
Labels:           ["none"]
Annotations:      ["none"]
Status:           Pending
IP:               
IPs:              ["none"]
Containers:
  critical-container:
    Image:      nginx
    Port:       ["none"]
    Host Port:  ["none"]
    Limits:
      memory:  3Gi
    Requests:
      cpu:        1
      memory:     3Gi
    Environment:  ["none"]
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ns8rk (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  kube-api-access-ns8rk:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              ["none"]
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  105s  default-scheduler  0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. no new claims to deallocate, preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.

## Consulta de eventos. Para buscar los fallos usamos grep Warning
kubectl events | grep Warning
49m                 Warning   InvalidDiskCapacity       Node/controlplane   invalid capacity 0 on image filesystem
14m (x3 over 15m)   Warning   Failed                    Pod/low-priority    Failed to pull image "ngix": failed to pull and unpack image "docker.io/library/ngix:latest": failed to resolve reference "docker.io/library/ngix:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
14m (x3 over 15m)   Warning   Failed                    Pod/low-priority    Error: ErrImagePull
14m (x3 over 15m)   Warning   Failed                    Pod/low-priority    Error: ImagePullBackOff
14m                 Warning   Failed                    Pod/low-priority    Error: ErrImagePull
14m                 Warning   Failed                    Pod/low-priority    Failed to pull image "ngix": failed to pull and unpack image "docker.io/library/ngix:latest": failed to resolve reference "docker.io/library/ngix:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
14m                 Warning   Failed                    Pod/low-priority    Error: ImagePullBackOff
4m1s                Warning   FailedScheduling          Pod/critical-app    0/1 nodes are available: 1 Insufficient cpu, 1 Insufficient memory. no new claims to deallocate, preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod.

## Obtener el yaml de critical-app
kubectl get pods critical-app -o yaml > critical-app.yaml

## El fichero original es muy grante y complejo, lo sustituimos por un pod más manejable
nano critical-app.yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-app
spec:
  containers:
  - name: nginx
    image: nginx
  priorityClassName: high-priority

## Borramos el pod critical-app, está obstruyendo la nueva importación
kubectl delete pods critical-app 
pod "critical-app" deleted from default namespace

## Improtamos el fichero.yaml
kubectl apply -f critical-app.yaml 
pod/critical-app created

## 38º El sistema de kubernetes permite expandirse y ofrecer más de dos planificadores para los pods, nodos, contenedores y personalizarlos. Cuando hay varios planificadores cada uno debe llevar distinto nombre para diferenciarse.

## Muestra del scheduler-config_1.yaml
nano scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default-scheduler

## Muestra del scheduler-config_2.yaml
nano my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler
leaderElection:
  leaderElect: true
  resourceNamespace: kube-system
  resourceName: lock-object-my-scheduler

## Muestra del scheduler-config_3.yaml
nano my-scheduler-2-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler-2

## Deplegar los nuevos planificadores
wget https://storage.googleapis.com/kubernetes-release/v1.XX.0/bin/linux/amd64/kube-scheduler

## Otra forma de establecer los planificadores es a través de un fichero-pod-scheduler.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --config=/etc/kubernetes/my-scheduler-config.yaml
     image: k8s.gcr.io/kube-scheduler-amd64:v1.XX.X
     name: kube-scheduler

## Localizar los pods del kube-system
kubectl get pods --namespace=kube-system
kubectl get pods --namespace kube-system
kubectl get pods -n kube-system 
NAME                                   READY   STATUS    RESTARTS   AGE
coredns-6f6c7df987-sr9kf               1/1     Running   0          12m
coredns-6f6c7df987-x929h               1/1     Running   0          12m
etcd-controlplane                      1/1     Running   0          13m
kube-apiserver-controlplane            1/1     Running   0          13m
kube-controller-manager-controlplane   1/1     Running   0          13m

## Crear el pod con el nuevo plainificador
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
  schedulerName: my-custom-scheduler

## importar el fichero
kubectl apply -f pod-definition.yaml

## Verificar el pod
kubectl get pods # --> Si hay un error el estado es Pending, si todo va bien estará en Running

## Evaluar si hay fallos
kubectl describe pod nginx

## Visualizamos el planificador elegido con el comando:
kubectl get events -o wide

## Los registros de eventos se guardan en:
kubectl logs my-custom-scheduler --namespace=kube-system

## Localizar la imagen del pod a través de la descripción y filtra el namespace
kubectl describe pod kube-controller-manager-controlplane -n kube-system | grep Image
    Image:         registry.k8s.io/kube-controller-manager:v1.35.0
    Image ID:      registry.k8s.io/kube-controller-manager@sha256:3e343fd915d2e214b9a68c045b94017832927edb89aafa471324f8d05a191111

## Obtener los serviceaccounts
kubectl get serviceaccounts 
NAME      AGE
default   18m

## Consultar los serviceaccounts del namespace y filtrar los que lleven my*
kubectl get serviceaccounts -n kube-system | grep my
my-scheduler    

## Consultar los clusterrolebinding y filtrar los que lleven my*
kubectl get clusterrolebinding | grep my
my-scheduler-as-kube-scheduler                                  ClusterRole/system:kube-scheduler                                                  5m16s
my-scheduler-as-volume-scheduler                                ClusterRole/system:volume-scheduler                                                5m16s

## Listar el directorio del usuario
ls -lh
total 16K
-rw-r--r-- 1 root root 336 Jul  7 17:41 my-scheduler-configmap.yaml
-rw-r--r-- 1 root root 155 Jun 17 11:24 my-scheduler-config.yaml
-rw-r--r-- 1 root root 893 Jun 17 11:24 my-scheduler.yaml
-rw-r--r-- 1 root root 105 Jun 17 11:24 nginx-pod.yaml

## Aplicar el fichero my-scheduler-configmap.yaml 
kubectl apply -f my-scheduler-configmap.yaml 
configmap/my-scheduler-config created

## Aplicar el fichero my-scheduler.yaml
kubectl apply -f my-scheduler.yaml 
pod/my-scheduler created

## Editar el fichero my-scheduler-pod.yaml y corregir la imagen defectuosa
nano my-scheduler-pod.yaml
apiVersion: v1
kind: Pod
metadata:
metadata:
  labels:
    run: my-scheduler
  name: my-scheduler
  namespace: kube-system
spec:
  containers:
  - name: nginx
    image: registry.k8s.io/kube-scheduler:v1.35.0 
#    image: registry.k8s.io/kube-controller-manager:v1.35.0 # This image isn't compatible to mission
  schedulerName: my-custom-scheduler

## Aplicar el fichero my-scheduler.yaml
kubectl apply -f my-scheduler.yaml 
pod/my-scheduler configured

## Editar el fichero nginx-pod.yaml y agregar el scheduleName
nano nginx-pod.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: nginx 
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-scheduler

## Aplicar el fichero nginx-pod.yaml
kubectl apply -f nginx-pod.yaml 
pod/nginx created

## 39.1º La plantilla de los pods se puede establecer para asignar la prioridad, el contenedor y el límite de recursos
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  priorityClassName: high-priority
  containers: data-processor
  - name: simple-webapp-color
    image: simple-webapp-color
    resources:
      requests:
        memory: "1Gi"
        cpu: 10

## 39.2º Este pod solo es válido en un nodo que lleve más de 10 CPU y más de 1Gi de RAM. Si hay más pods en la cola de programación deben esperar a que se les asigne los nodos. Cada uno tiene una prioridad distinta que organiza a través de alta/baja prioridad, el pod que lleva la prioridad alta y se incluye un fichero para configurarla.

nano priority-class-definition.yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "The priority class should be used for XYZ service pods only"

## 39.3º Las fases del filtro de prioridades establece un orden para la llegada de los pods al nodo.
## 1º --> LLegan a la cola de programación. Se reordena para reubicar según la prioridad de acceso.
## 2º --> LLega al filtro para descartar los nodos incompatibles con el pod
## 3º --> LLega a la fase de puntuaciones para evaluar los nodos, el nodo con más puntuación es el elegido para usar el pod
## 4º --> En la fase de vinculación se termina el proceso de selección y empieza a trabajar el pod

## 39.4º Los plugins/complementos ayudan al sistema con sus funciones, el sistema de filtros de prioridad lleva un complemento en cada fase.
## 1º --> Scheduling Queue lleva el PrioritySort
## 2º --> Filtering lleva el NodeResourcesFit, NodeName, NodeUnschedulable, NodeResourcesFit, TaintToleration, NodePorts, NodeAffinity 
## 3º --> Scoring lleva el NodeResourcesFit, ImageLocaly, NodeResourcesFit, TaintToleration y NodeAffinity
## 4º --> Binding lleva el DefaultBinder

## 39.5º A parte de los complementos de sistema se puede añadir más complementos y personalizarlos. Los personalizables son:
## 1º --> queueSort
## 2º --> prefilter, filter, postfilter
## 3º --> preScore, score, reserve
## 4º --> permit, prebind, bind, post,bind

## 39.6º Los programadores pueden más de dos perfiles dentro para trabajar en conjunto:
nano my-scheduler-2-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler-2
- schedulerName: my-scheduler-3
- schedulerName: my-scheduler-4
# Este fichero de configurar el programador lleva tres programadores distintos, disponible desde Kubernetes v1.18
#-----------------------------------------
nano my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler

#-------------------------------------------------
nano scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: default

## 39.7º Dentro del planificador del fichero de programación se puede establecer los complementos que se quieren configurar:
nano my-scheduler-2-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
- schedulerName: my-scheduler-2
  plugins:
    score:
      disabled:
       - name: TaintToleration
      enabled:
       - name: MyCustomPluginA
       - name: MyCustomPluginB
- schedulerName: my-scheduler-3
  plugins:
    preScore:
      disabled:
       - name: "*"
    score:
      disabled:
       - name: "*"
- schedulerName: my-scheduler-4
## el planificador 2 y 3 llevan un plugin personalizado para la puntuación con el estado de activado y desactivado

## 40.1º El servicio del controlador de admininstración evalua las opciones de seguirad disponibles y verificar si se puede crear un pod.

## 40.2º El fichero .kube/config contiene un yaml con el certificado para autorizar el cluster y el kubelet lee el contenido del fichero.

## 40.3º Los procesos de creación de pods empiezan desde el kubelet y si es válido el certificado sigue a la autentificación, la autorización de los recursos y crear el objeto de destino:
## 1º --> El kubelet lee los datos de seguridad, el estado de los recursos y valida el servicio.
## 2º --> Se autentifica el controlador y el objeto
## 3º --> Se autoriza el acceso al sistema después de autentificar el kubelet.
## 4º --> El controlador de admisión aplica las medidas de seguridad para evitar el acceso no autorizado al cluster.
## 5º --> Crea el objeto que se quiere trabajar y alojar.

## Muestra de un fichero.yaml con un rol de autorización, debe poder permitir: listar, consultar, crear, actualizar y borrar. Si no está autorizado se rechaza la operación:
nano developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","get","create","update","delete"]

## Este fichero solo ofrece la creación de pods en los namespaces determinados
nano developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create"]
  resourceNames: ["blue","orange"]

## Fichero yaml con el pod, puede recibir restricciones
nano web-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
   containers:
      - name: ubuntu
        iamge: ubuntu:latest
        command: ["sleep" , "3600"]
        securityContext:
          runAsUser: 0
          capabilities:
            add: ["MAC_ADMIN"]

## Las restricciones que pueden impedir crear el pod son:
## 1º Solo permite imagenes registradas
## 2º No permite acceder con root
## 3º Solo permite capacidades registradas
## 4º El pod tiene siempre el mismo label

## Dentro de Kubernetes hay preconfigurados varios tipos de controladores de admisión:
## 1º --> AlwaysPullImages: Cuando se va a crear el pod inicia la imagen necesaria.
## 2º --> DefaultStorageCluster: Ofrece siempre el almacenamiento estandar al pod.
## 3º --> EventRateLimit: Limita las solicitudes al servidor api para no atascarlo, este servicio puede tomar más de una órden a la vez.
## 4º --> NamespaceExists: Verifica si el nombre del namespace está presente, normalmente si no existe se bloquea el desarrollo pod, pero se puede modificar para crearlo en el mismo momento del pod.
## 5º --> Los 4 primeros están activados por defecto, pero hay uno que está desactivado: NamespaceAutoProvision; este controlador se encarga de crear un namespace para superar el obstáculo por ausencia de ns.

## Crea un pod con un namespace no declarado:
kubectl run nginx --image nginx --namespace blue
## Si el namespace no está presente debe impedir la creación del pod con el mensaje; ns_not_found. El controlador de admisión ha evaluado el pod y solo superó las pruebas del kubelet, la autenticación y la autorización, el verificados de los ns lo bloqueó.

## Comando para consultar los controladores activos
kube-apiserver -h | grep enable-admission-plugins

## Comando para consultar los controladores activos con comando extenso
kubectl exec kube-apiserver-controlplane -n -- kube-sytem -- kube-apiserver -h | grep enable-admission-plugins

## Activamos el módulo de admisión para permitir la creación automática de los ns:
nano kube-apiserver.service
--enable-admission-plugins=NodeRestriction,NamespaceAutoProvision

## Creamos un fichero.yaml que permite acceder a la nueva opción de crear los ns desantendidos:
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --authorization-mode=Node,RBAC
    - --advertise-address=172.17.0.107
    - --allow-privileged=true
    - --enable-bootstrap-token-auth=true
    - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
    image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
    name: kube-apiserver

## Después de admitir esta opción, los pods que lleven el namespce sin crear, el sistema los añade sin intervención del usuario.
kubectl run nginx --image nginx --namespace blue
kubectl get namespaces

## Listar los ficheros de kubernetes con el comando find y ls unidos
find /etc/kubernetes/imgvalidation/ -type f -name "*.yaml" -exec ls -hl {} \+
-rw-r--r-- 1 root root 164 Jul  8 15:41 /etc/kubernetes/imgvalidation/admission-configuration.yaml
-rw-r--r-- 1 root root 145 Jul  8 15:41 /etc/kubernetes/imgvalidation/imagepolicy-conf.yaml
-rw-r--r-- 1 root root 494 Jul  8 15:41 /etc/kubernetes/imgvalidation/kubeconf.yaml

## 41.1º Los controladores también tienen la opción de realizar mutaciones a los ficheros y parámetros de configuración de los objetos, algunos atributos aunque no se escriben en el fichero, el sistema los añade a través de las mutaciones en los controladores de admisión. El NamespaceAutoProvision realiza una mutación para crear un objeto que no se creó anteriormente y NameExists valida la operación en curso.

## 41.2º Hay dos controladores especiales llamados MutantingAdmissionWebhook y validantAdminssionWebhook. Se puede configurar para que apunte a un servidor interno de Kubernetes o apuntar a un servidor externo.

## 1º Creamos un servidor de admisión Webhook. La plantilla se descarga con este enlace
https://github.com/kubernetes/kubernetes/blob/v1.13.0/test/images/webhook/main.go

## 2º Configuramos el servidor de admisión a través de un fichero.yaml
nano webhook-service.yaml
apiVersion: adminissionregistration.k8s.io/v1
kind: Service
metadata:
   name: "pod-policy.example.com"
webhooks:
- name: "pod-policy.example.com"
  clientConfig:
    service:
      namespace: "webhook-namespace"
      name: "webhook-service"
    caBundle: "Ci0tLS0Qk....tLS0K"
  rules:
  - apiGroups:   [""]
    apiVersions: ["v1"]
    operations:  ["CREATE"]
    resources:   ["pods"]
    scope:       "Namespaced"

## 41.3º A parte de los controladores internos también se puede crear los controladores a petición del usuario.

## Filtrar los ficheros por la palabra clave --> runAsNonRoot:
grep runAsNonRoot *.yaml
pod-with-conflict.yaml:    runAsNonRoot: true
pod-with-override.yaml:    runAsNonRoot: false
webhook-deployment.yaml:        runAsNonRoot: true

## Filtrar los ficheros por la palabra clave --> securityContext:
grep securityContext *.yaml
pod-with-conflict.yaml:# A pod with a conflicting securityContext setting: it has to run as a non-root
pod-with-conflict.yaml:  securityContext:
pod-with-defaults.yaml:# A pod with no securityContext specified.
pod-with-override.yaml:# A pod with a securityContext explicitly allowing it to run as root.
pod-with-override.yaml:  securityContext:
webhook-deployment.yaml:      securityContext:

## Crea un pod con busybox y un comando para usar un usuario concreto
nano pod-with-defaults.yaml 
# A pod with no securityContext specified.
# Without the webhook, it would run as user root (0). The webhook mutates it
# to run as the non-root user with uid 1234.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-defaults
  labels:
    app: pod-with-defaults
spec:
  restartPolicy: OnFailure
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]

##
cat pod-with-override.yaml 
# A pod with a securityContext explicitly allowing it to run as root.
# The effect of deploying this with and without the webhook is the same. The
# explicit setting however prevents the webhook from applying more secure
# defaults.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-override
  labels:
    app: pod-with-override
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: false
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]

##
cat pod-with-conflict.yaml 
# A pod with a conflicting securityContext setting: it has to run as a non-root
# user, but we explicitly request a user id of 0 (root).
# Without the webhook, the pod could be created, but would be unable to launch
# due to an unenforceable security context leading to it being stuck in a
# 'CreateContainerConfigError' status. With the webhook, the creation of
# the pod is outright rejected.
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-conflict
  labels:
    app: pod-with-conflict
spec:
  restartPolicy: OnFailure
  securityContext:
    runAsNonRoot: true
    runAsUser: 0
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "echo I am running as user $(id -u)"]
