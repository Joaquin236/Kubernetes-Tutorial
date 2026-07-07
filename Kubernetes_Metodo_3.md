# IMPLEMENTACIÓN ALTERNATIVA
# Video de origen del Método 3 --> https://www.youtube.com/watch?v=DCoBcpOA7W4&t=168s
# Documentación 1: https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download
# Documentación 2: https://kubernetes.io/docs/concepts/services-networking/ingress/
# Documentación 3: https://kind.sigs.k8s.io
# Documentación 4: https://kubernetes.io/docs/reference/kubectl/conventions/
# Documentación 5: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands
# Curso de pago Udemi --> https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/?couponCode=MT260629G3
# Curso gratis con simulador de Kubernetes --> https://learn.kodekloud.com/user/dashboard
# Curso gratis Google Skills --> https://www.skills.google/course_templates/2?locale=es
# Conclusión: Este método es el que más éxito tiene por ahora
# 1º Descargamos el minikube
curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64

# 2º Iniciamos el minikube, no usar el usuario '#', este comando solo funciona con '$'
minikube start

# 3º Accedemos al cluster, mostramos información y acceder al servidor con dashboard
kubectl get po -A
minikube dashboard

# 4.1º Crea una muestra de desarrollo y exponer en el puerto 8080:
kubectl create deployment hello-minikube --image=kicbase/echo-server:1.0
kubectl expose deployment hello-minikube --type=NodePort --port=8080
# Esto tomará un tiempo, pero el desarrollo mostrará cuando puedas iniciarlo:

kubectl get services hello-minikube
# La forma más fácil de acceder al servicio es iniciando el minikube con el navegador web:

minikube service hello-minikube
# Alternativamente se puede declarar el puerto:

kubectl port-forward service/hello-minikube 7080:8080
# Excelente, tu aplicación estará accesible con la URL http://localhost:7080/.

# Prodrás ver la respuesta de la aplicación en la salida de la consola. Intenta cambiar la ruta y observa los cambios.
# Puedes pasar un parámetro por POST y obervar la salida.

# 4.2º Para acceder al desarrollo del LoadBalancer, usa el comando 'minikube tunnel':

kubectl create deployment balanced --image=kicbase/echo-server:1.0
kubectl expose deployment balanced --type=LoadBalancer --port=8080
# En otra consola de comandos, inicia el tunnel para crear una ruta IP para el desarrollo del balanced:

minikube tunnel
# Para buscar la ruta IP, inicia este comando y verifica la columna de IP_Externa:

kubectl get services balanced
# Your deployment is now available at <EXTERNAL-IP>:8080

# 4.3º Inicia el complemento INGRESS:
minikube addons enable ingress
/*
The following example creates simple echo-server services and an Ingress object to route to these services.

kind: Pod
apiVersion: v1
metadata:
  name: foo-app
  labels:
    app: foo
spec:
  containers:
    - name: foo-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: foo-service
spec:
  selector:
    app: foo
  ports:
    - port: 8080
---
kind: Pod
apiVersion: v1
metadata:
  name: bar-app
  labels:
    app: bar
spec:
  containers:
    - name: bar-app
      image: 'kicbase/echo-server:1.0'
---
kind: Service
apiVersion: v1
metadata:
  name: bar-service
spec:
  selector:
    app: bar
  ports:
    - port: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
    - http:
        paths:
          - pathType: Prefix
            path: /foo
            backend:
              service:
                name: foo-service
                port:
                  number: 8080
          - pathType: Prefix
            path: /bar
            backend:
              service:
                name: bar-service
                port:
                  number: 8080
---
Apply the contents
*/

kubectl apply -f https://storage.googleapis.com/minikube-site-examples/ingress-example.yaml
# Espera a la dirección del kubernetes Ingress:

kubectl get ingress
/*
NAME              CLASS   HOSTS   ADDRESS          PORTS   AGE
example-ingress   nginx   *       <your_ip_here>   80      5m45s
Note for Docker Desktop Users:
To get ingress to work you’ll need to open a new terminal window and run minikube tunnel and in the following step use 127.0.0.1 in place of <ip_from_above>.

Now verify that the ingress works
*/

$ curl <ip_from_above>/foo
# Request served by foo-app
# ...

$ curl <ip_from_above>/bar
# Request served by bar-app
# ...

## 5º Pausar el minikube sin afectar a los servicios activos:
minikube pause

# Reanudar minikube en pausa:
minikube unpause

# Detener el cluster:
minikube stop

# Cambiar el límite de memoria (Requiere reinicio):
minikube config set memory 9001

# Explorar catalogo de complementos disponibles:
minikube addons list

# Crea un segundo cluster con una versión anterior:
minikube start -p aged --kubernetes-version=v1.34.0

# Borra todos los cluster activos:
minikube delete --all

# 6.1º Añadir tres nodos, (1 Nodo Maestro y 2 Nodos Trabajadores):
minikube start --nodes=3 --driver=docker

# 6.2º Añadir un nodo trabajdor:
minikube node add --worker

# 7.1º Añadir los pods de forma automática:
kubectl run test --image=nginx

## También se puede añadir a otro namespace si fuese necesario
kubectl run redis --namespace finance --image redis 
pod/redis created
## Este comando ofrece un label personalizado
kubectl run redis redis --image=redis:alpine --labels=tier=db
pod/redis created

# 7.2º Crear un pod a través de un fichero pod.yaml, para aplicarlo se enlaza el comando kubectl con el fichero:
nano pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test2
spec:
  containers:
  - name: web
    image: nginx

kubectl apply -f pod.yaml

# 8º Editamos el pod con kubectl edit pod <Nombre_pod>:
kubectl edit pod test

# 9º A través del parámetro -o wide que ofrece el visor de pods, ofrece más información.
kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
test    1/1     Running   0          108m   10.244.0.8   minikube   <none>           <none>
test2   1/1     Running   0          101m   10.244.0.9   minikube   <none>           <none>

kubectl get pods test -o yaml # --> ofrece descipciones con formato yaml
kubectl describe pod test # --> ofrece descipciones con formato amigable

# Si añadimos "<comando> | less" podemos navegar por la salida del comando

# 10º Con el parámetro logs verificamos el registro de estado del pod
kubectl logs test
kubectl logs test2

# 11.1º Borramos el pod test
kubectl delete pod test
pod "test" deleted from default namespace

# 11.2º Editamos el fichero pod.yaml anterior y cambiamos el nombre y el tipo de imagen en ejecución:
nano pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test 
spec:
  containers:
  - name: web
    image: nginx:alpine

kubectl apply -f pod.yaml 
pod/test created

# 11.3º Si editamos un pod existente y aplicamos debe aparecer como configurado;
nano pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - name: web
    image: nginx

kubectl apply -f pod.yaml 
pod/test configured

kubectl get nodes ; kubectl get pods
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   13h   v1.35.1
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (37s ago)   6m9s
test2   1/1     Running   0             155m

# 12.1º Para obtener los namespaces, usamos el comando:
kubectl get namespaces
NAME                   STATUS   AGE
default                Active   13h
kube-node-lease        Active   13h
kube-public            Active   13h
kube-system            Active   13h
kubernetes-dashboard   Active   13h
# Los namespaces default y/o creados manualmente son los que tenemos que modificar.
# Los namespaces creados por el sistema suelen llevar contenido crítico y deben permanecer sin cambios.

# 12.2º También se puede obtener los pods de cualquier namespace
kubectl --namespace kube-system get pods
NAME                               READY   STATUS    RESTARTS        AGE
coredns-7d764666f9-jr4pq           1/1     Running   1 (12h ago)     13h
etcd-minikube                      1/1     Running   1 (12h ago)     13h
kube-apiserver-minikube            1/1     Running   1 (12h ago)     13h
kube-controller-manager-minikube   1/1     Running   4 (3h11m ago)   13h
kube-proxy-7rxmx                   1/1     Running   1 (12h ago)     13h
kube-scheduler-minikube            1/1     Running   1 (12h ago)     13h
storage-provisioner                1/1     Running   3 (3h10m ago)   13h

## Obtener los pods de otros namespaces
kubectl get pods --namespace research
NAME    READY   STATUS             RESTARTS       AGE
dna-1   0/1     CrashLoopBackOff   6 (107s ago)   7m35s
dna-2   0/1     CrashLoopBackOff   6 (114s ago)   7m35s

# 12.3 Creamos un nuevo namespace donde podamos editar
kubectl create ns development
namespace/development created

# 12.4 Si el siguiente pod lo queremos llevar a otro namespace, necesitamos establecerlo durante la creación del pod:
kubectl -n development run nginx --image=nginx

kubectl -n development get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          29s

kubectl -n default get pods
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (41m ago)   46m
test2   1/1     Running   0             3h16m

## 12.5 Si necesitamos borrar el namespace, borrará todo lo que contiene
kubectl -n development get pods ; kubectl delete ns development ; kubectl -n development get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          30m
namespace "development" deleted
No resources found in development namespace.

## Al buscar el namespace borrado ya no se muestra
kubectl get namespaces
NAME                   STATUS   AGE
default                Active   14h
kube-node-lease        Active   14h
kube-public            Active   14h
kube-system            Active   14h
kubernetes-dashboard   Active   14h

## 12.6º Este comando mostrará la configuración del cluster activo
kubectl config view

## 12.7º Estos valores pueden definirse en el pod manualmente
kubectl config set-context minikube --namespace=development
Context "minikube" modified.

kubectl get pods
No resources found in development namespace.

kubectl get ns
NAME                   STATUS   AGE
default                Active   14h
development            Active   7m14s
kube-node-lease        Active   14h
kube-public            Active   14h
kube-system            Active   14h
kubernetes-dashboard   Active   14h

## Después de los cambios todo lo nuevo va a development
kubectl config view

kubectl -n default get pods
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (97m ago)   102m
test2   1/1     Running   0             4h11m
kubectl apply -f pod.yaml 
pod/test created

kubectl -n default get pods
NAME    READY   STATUS    RESTARTS      AGE
test    1/1     Running   1 (97m ago)   103m
test2   1/1     Running   0             4h12m

kubectl -n development get pods
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          30s

## Para revertir cambios solo necesitamos el mismo comando con el valor inicial
kubectl config set-context minikube --namespace=default
Context "minikube" modified.

## 13.1º Creamos una estrcutura para el deployment
nano deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        ports:
        - containerPort: 80
        
kubectl apply -f deployment.yaml 
deployment.apps/nginx-deployment created

kubectl get nodes ; kubectl get pods
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   19h   v1.35.1
NAME                                READY   STATUS    RESTARTS        AGE
nginx-deployment-75fdcbbc74-lm2wj   1/1     Running   0               7m20s
nginx-deployment-75fdcbbc74-sqrwk   1/1     Running   0               7m20s
nginx-deployment-75fdcbbc74-zb47w   1/1     Running   0               7m20s
test                                1/1     Running   2 (3h59m ago)   5h59m
test2                               1/1     Running   1 (3h59m ago)   8h

kubectl get deployments.apps
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           10m

## Estos comandos muestran que el deployment está operativo tras crearlo

## 13.2º Con el comando kubectl create se puede crear el deploy con un diseño manual
kubectl create deployment nginx-cli --image=nginx:alpine

kubectl get deployments.apps
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-cli          0/1     1            0           19s
nginx-deployment   3/3     3            3           16m

## 13.3º El fichero deployment.yaml se puede modificar para realizar un escalado y adaptación de los servicios
nano deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 5
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
        image: nginx:alpine
        ports:
        - containerPort: 80
kubectl apply -f deployment.yaml
deployment.apps/nginx-deployment configured
El sistema identifica cuando hay que crear, modificar o borrar

## 13.4º Si necesitamos editar un deploy que está en ejecución usamos:
kubectl edit deployment.apps ngingx-cli
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2026-07-01T15:38:56Z"
  generation: 1
  labels:
    app: nginx-cli
  name: nginx-cli
  namespace: default
  resourceVersion: "18546"
  uid: d282071a-79a6-4d73-b95c-2cf3f726520e
spec:
  progressDeadlineSeconds: 600
  replicas: 1 --> 3
  revisionHistoryLimit: 10
  selector:  
    matchLabels:
      app: nginx-cli
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:  
    metadata:
      labels:
        app: nginx-cli
La línea de replicas empezó con 1, lo ampliamos a 3
kubectl edit deployments.apps nginx-cli
deployment.apps/nginx-cli edited

## 13.5 Si la carga de trabajo cambia y necesitamos reducir el número de replicas se realizará una escalada decremental:
kubectl scale deployment nginx-deployment --replicas=2

## 13.6 En caso de necesitar una recuperación del servicio, verificar las versiones, usaremos:
kubectl rollout history deployment nginx-deployment
deployment.apps/nginx-deployment 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
## Para confirmar la versión anterior usaremos:
kubectl rollout undo deployment nginx-deployment --to-revision=3

## 13.7 Para borrar necesitamos:
kubectl delete node minikube
node "minikube" deleted

kubectl delete deployment.apps nginx-cli
deployment.apps "nginx-cli" deleted from default namespace

kubectl delete deployment.apps nginx-deployment
deployment.apps "nginx-deployment" deleted from default namespace

kubectl delete pod webapp
webapp pod deleted

## 14.1º Si necesitamos crear un set de réplicas, usaremos esta estructura en fichero.yaml
nano replicaset.yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: myapp-replicaset
 labels:
    app: myapp
    type: Front-end
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

kubectl create -f replicaset.yaml

## 14.2º Consultar el set de réplicas activas:
kubectl get replicaset

## 14.3º Borrar el set de réplicas activas, borrará todo el contenido del set;
kubectl delete replicaset myapp-replicaset

## 14.4º Para reemplazar el set usaremos:
kubectl replace -f replicaset.yaml

## 14.5º Para escalar y mejorar el rendimiento del set aplicacos este comando:
kubectl scale --replicas=5 -f new-replica-set.yaml 

## 15º La consola puede construir el fichero yaml completo de los PODs con el comando:
kubectl run nginx --image=nginx --dry-run=client -o yaml
## Genera un manifiesto del Pod con un fichero yaml (-o yaml). No crearlo de inmediato (--dry-run)
## Para crear el POD creado debe usar el comando:
kubectl create -f <name_file.yaml>

## 16º El despliegue de los deployment también disponen del mismo comando:
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml
## El manifiesto del Deployment también debe realizarse manualmente:
kubectl apply -f <name_file.yaml>

## 17º Este comando redirige la salida de la interfaz de consola a un fichero nuevo. Si existe sobreescribe su contenido:
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
## En las versiones más modernas de Kubernetes admite personalizar el número de replicas a trabajar:
kubectl create deployment --image=nginx nginx --dry-run=client --replicas=4 -o yaml > nginx-deployment.yaml

## 18º Los servicios ofrecen interacción con los pods a través del comando curl <http://IP:puerto>
## Las direcciones IP de los pods no son enrutables con la dirección del host. Impidiendo acceder desde la navegación tradicional
## Dentro del nodo se encuentra un servicio que conecta el host con los contenedores activos
## Tipos de servicios: ["Node_Port" , "Cluster_IP" , "Load_Balancer"]
## Node_Port --> Cada contenedor lleva el puerto con su dirección con otra CIDR, el servicio hace de enlace entre el host y el nodo, este servicio tiene su dirección diferente a la del contenedor.
## Fichero yaml del servicio NodePort;
apiVersion: v1
kind: Service
metadata:
    name: myapp-service
spec:
    type: NodePort
    ports:
     - targetPort: 80
       port:80
       nodePort: 30008
    selector:
       app: myapp
       type: front-end 
## Fichero yaml del pod asociado al servicio, los parámetros del labels se asocian con el yaml del servicio:
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
 labels:
    app: myapp
    type: front-end
## Comando para crear el servicio
kubectl creade -f <file_name.yaml>
## Comando para consultar los servicios:
kubectl get services
## Después de configurarlo, el navegador GUI del cliente puede visitar el contenido del contenedor que queremos visitar
## Cuando hay más de un contenedor, el nodo reparte el servicio entre los contenedores. Cuando hay más de un cluster, el servicio se expande a los demás clusters
## En el segundo ejemplo de los node_ports, se ofrecen más de dos direcciones para enlazarse con los servidores. Si se produce un cambio, el servidor de nodos cambia la estructura sin necesidad de actuar

## 19º En el diseño del Cluster_IP los servidores de los contenedores se agrupan por sus funciones:
## Front-end --> Back-end --> Redis. Cada uno tiene la IP interna del nodo
## Fichero del Servicio ClusterIP:
apiVersion: v1
kind: Service
metadata:
     name: back-end
spec:
    type: ClusterIP
    ports:
     - targetPort: 80
       port: 80
    selector:
       app: myapp
       type: back-end

## Fichero del Pod asociado al servicio;
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
 labels:
    app: myapp
    type: back-end
spec:
  containers:
  - name: nginx-container
    image: nginx

## 20º El servicio Balanced depende de un sistema Cloud compatible con él y una aplicación de MV, si se usa en un entorno no compatible es como un NodePort
## Fichero yaml del LoadBalancer:
apiVersion: v1
kind: Service
metadata:
    name: myapp-service
spec:
    type: LoadBalancer
    ports:
     - targetPort: 80
       port: 80
       nodePort: 30008
-----------------------------------------------------------------------
## Fichero NodePort_2:
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  ports:
  - nodePort: 30080
    port: 8080
    targetPort: 8080
  selector:
    name: simple-webapp
  type: NodePort
------------------------------------------------------------------------
kubectl create -f service-definition-1.yaml 
service/webapp-service created

kubectl get service
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes       ClusterIP   10.43.0.1       <none>        443/TCP          30m
webapp-service   NodePort    10.43.138.146   <none>        8080:30080/TCP   2m28s

curl http://10.43.138.146:8080
<!doctype html>
<title>Hello from Flask</title>
<body style="background: #e74c3c;"></body>
<div style="color: #e4e4e4;
    text-align:  center;
    height: 90px;
    vertical-align:  middle;">
  <h1>Hello from simple-webapp-deployment-6dcd5bb6b6-lgd7t!</h1>
</div>
---------------------------------------------------------------------------------
## Comando imperativo para crear un servicio:
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml # --> crea los pods sin intervención
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml # --> los pods hay que seleccionarlos manualmente
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml # --> crea los valores de los pods pero no puedes elegir el puerto a mano
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml # --> no puedes usar los selectores ni los letreros

## 21º Cuando se usa el comando curl obtenemos:
curl http://10.22.0.13:8080
<!DOCTYPE html>
<!--[if lt IE 7]>      <html lang="en" ng-app="myApp" class="no-js lt-ie9 lt-ie8 lt-ie7"> <![endif]-->
<!--[if IE 7]>         <html lang="en" ng-app="myApp" class="no-js lt-ie9 lt-ie8"> <![endif]-->
<!--[if IE 8]>         <html lang="en" ng-app="myApp" class="no-js lt-ie9"> <![endif]-->
<!--[if gt IE 8]><!--> <html lang="en" ng-app="myApp" class="no-js"> <!--<![endif]-->
<head>
  <meta charset="utf-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <title>Connectivity Test</title>
  <meta name="description" content="">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="bower_components/html5-boilerplate/dist/css/normalize.css">
  <link rel="stylesheet" href="bower_components/html5-boilerplate/dist/css/main.css">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css">
  <link rel="stylesheet" href="css/blue.css">
  <link rel="stylesheet" href="css/animation.css">
  <!--<link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" crossorigin="anonymous">-->
  <!--<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css" crossorigin="anonymous">-->
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
  <link href="https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css" rel="stylesheet" />

  <script src="bower_components/html5-boilerplate/dist/js/vendor/modernizr-2.8.3.min.js"></script>
</head>
<body>
  <!--<ul class="menu">-->
    <!--&lt;!&ndash;<li><a href="#!/view1">view1</a></li>-->
    <!--<li><a href="#!/view2">view2</a></li>&ndash;&gt;-->
  <!--</ul>-->

<!--  <style>
        body {
            background-image: url(' ../images/under_water.png ');
        }
  </style>-->

  <h3 style="text-align: center;
    color: white;"> Blue - Marketing Application </h3>

  <!--[if lt IE 7]>
      <p class="browsehappy">You are using an <strong>outdated</strong> browser. Please <a href="http://browsehappy.com/">upgrade your browser</a> to improve your experience.</p>
  <![endif]-->

  <div ng-view></div>

  <script src="https://code.jquery.com/jquery-3.3.1.min.js" crossorigin="anonymous"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
  <!--<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>-->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js"></script>
  <script src="bower_components/angular/angular.min.js"></script>
  <script src="bower_components/angular-route/angular-route.min.js"></script>
  <script src="app.js"></script>
  <script src="view1/view1.js"></script>
  <script src="components/version/version.js"></script>
  <script src="components/version/version-directive.js"></script>
  <script src="components/version/interpolate-filter.js"></script>

</body>
</html>

## 22.1º Cuando necesitemos ubicar los Pods en otro namespace escribimos --namespace=<name>:
kubectl create -f <file_name.yaml> --namespace=<name>
## También se pude establecer el namespace desde el fichero.yaml:
nano name_file.yaml
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
 namespace: dev
 labels:
    app: myapp
    type: front-end
spec:
  containers:
  - name: nginx-container
    image: nginx
kubectl create -f <file_name.yaml>

## 22.2º Los namespaces admiten ser creados desde un fichero.yaml:
nano name_file.yaml
apiVersion: v1
kind: Namespace
metadata:
 name: dev
kubectl create -f name_file.yaml

## 22.3º Cuando necesitamos entrar en el namespace default, no declaramos el namespace:
kubectl get pods
## Cuando necesitamos entrar en cualquier otro namecespace necesitamos declararlo:
kubectl get pods --namespace=<another_namespaces>

## 22.4º Si necesitamos cambiar el namespace predeterminado por otro usaremos:
kubectl config set-context $(kubectl config current-context) --namespace=dev

## Al cambiar el valor el comando 'kubectl get pods' ya no ofrecerá el default, ofrecerá el 'dev' como predeterminado, para ver el default necesitamos el --namespace=default:
kubectl get pods
kubectl get pods --namespace=default
kubectl get pods --namespace=pr

## Si queremos mostrar todos los pods en los namespaces completos usamos:
kubectl get pods --all-namespaces

## 22.5º Para evitar la saturación del servidor host de las unidades virtuales, se puede establecer cuotas de limitación a través de un fichero yaml:
nano <quote_file.yaml>
apiVersion: v1
kind: ResourceQuota
metadata:
 name: compute-quota
 namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 4Gi
    limits.cpu: "2"
    limits.memory 4Gi

kubectl create -f <quote_file.yaml>

## 22.6º Los clientes usarán la función de conexión mysql.connect() de sus aplicaciones de consultas para acceder al contenedor con bases de datos:
mysql.connect("db-service.dev.svc.cluster.local")
("Service_Name".Namespace.Service.Cluster_Domain)

## 23º Los modos de trabajo en kubernetes pueden ser:
### Impertativos: Se usa los comandos en primer plano, es más rápido pero más exigente con las órdenes
### Declarativos: Se usa los ficheros y rutas absolutas que indican la ubicación lógica de los ficheros para trabajar

## 24.1º Si necesitamos consultar los recursos disponibles de la api de kubernetes, usaremos:
kubectl api-resources
## 24.2º Para profundizar en la consulta de recursos, usaremos:
kubectl explain <resource> # --> modo normal
kubectl explain <resource>.spec # --> modo intermedio
kubectl explain <resource> --recursive # --> modo avazado
kubectl explain <resource>.spec --recursive # --> modo completo

## 25.1º Modo declarativo de creación de POD:
kubectl run redis --image=redis:alpine --dry-run=client -o yaml > redis-pod.yaml

cat redis-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis:alpine
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}

kubectl apply -f redis-pod.yaml

## 25.2º Modo imperativo sin fichero:
kubectl run redis --image=redis:alpine -l tier=db
pod/redis created

## 25.3º Crear un servicio para un pod y con puerto 6379:
kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP
service/redis-service exposed

## 25.4º Crear un deployment con 3 replicas y con imagen kodekloud:
kubectl create deployment webapp --replicas=3 --image=kodekloud/webapp-color
deployment.apps/webapp created

## 25.5º Crear un pod con puerto 8080:
kubectl run custom-nginx --image=nginx --port=8080
pod/custom-nginx created

## 25.6º Crear un namespace nuevo:
kubectl create namespace dev-ns
namespace/dev-ns created

## 25.7º Crear un deploy con el enlace a un namespace:
kubectl create deployment redis-deploy --replicas=2 -n dev-ns --image=redis
deployment.apps/redis-deploy created

## 25.8º Crear un pod y un servicio con los nombres httpd, el comando --expose enlaza el pod con el servicio, si no existe lo crea sin error de comando:
kubectl run httpd --port=80 --expose --image=httpd:alpine
service/httpd created
pod/httpd created

## 26º Diferentes formas de aplicar un fichero.yaml:
kubectl apply -f nginx.yaml
kubectl apply -f /path/to/file_nginx.yaml
kubectl apply -f nginx.yaml # --> si lo repites después de lanzarlo acutaliza el objeto creado
## Normalmente el fichero manual suele ser más pequeño que el fichero en vivo con el que trabaja kubernetes
## Existe un proceso intermedio donde el sistema lo interpreta en fichero.json
## 1º-> Local_file.yaml=yaml --> 2º-> Last Apply_Config.json=json --> 3º-> Kubernetes_Live.yaml=yaml

## 27.1º Programar el planificador back-end: en caso que no queremos el pod en el nodo genérico por falta de seguridad, en el fichero.yaml declaramos la clave 'nodeName' con el valor 'node02', una vez creado el sistema elige el nodo correspondiente, los pods que no cumplen con el valor del nodo no se aplicará, después identifica el nodo y lo asigna. Cuando no hay planificador (Scheduler) en la casilla de estado se informa que está el pod en modo 'Pendiente' y no se podrá usar el pod. Otro caso que puede suceder es el pod ya está creado y funcionando pero se le quiere cambiar el nodo, el sistema no admite cambiar la propiedad del nodo de un pod, la solución es crear un objeto vinculante y enviar la solicitud, es necesario usar la sintaxis de los objetos JSON para esta acción.

## 27.2º Sintaxis del fichero yaml con el nameNode declarado:
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
  nodeName: node

## 27.3º Cambiamos el nodo sin número por node02:
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
  nodeName: node02

## 27.4º Con el comando curl creamos un objeto y lleva los valores con JSON:
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
 name: nginx
 labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 8080
  nodeName:  

nano pod-bind-definition.yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node2
(Estos valores se guardan en el comando con JSON)

## 27.5º Comando curl completo, este comando debe ser modificado para adaptarlo a la situación;
curl --header "Content-Type:application/json" --request POST --data '{"apiVersion":"v1", "kind":"Binding","metadata":{"name":"nginx"},"target":{"apiVersion":"v1","kind":"Node","name":"node2"}}' http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding # --> Este comando no parece funcionar por ahora

## 27.6º Esta plantilla lleva el atributo para elegir manualmente el nombre del nodo:
nano nginx.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  nodeName: <Name _ of _ node>
  containers:
  -  image: nginx
     name: nginx

## 28.1º Plantilla con los identicadores de Aplicacción y Función:
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
 name: simple-webapp
 labels:
   app: App1
   Function: Fron-end
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
      - containerPort: 8080

## 28.2º Filtramos los pods por aplicación a través del comando:
kubectl get pods --selector app=App1

## Comando para filtrar por ns:
kubectl get pods --namespace kube-system 

## 28.3º Plantilla de replicaset con la aplicación y función:
nano replicaset.yaml

apiVersion: apps/v1
kind: ReplicaSet
metadata:
 name: simple-webapp
 labels:
    app: App1
    type: Front-end
spec:
 template:
    metadata:
    replicas: 3
    selector:
      matchLabels:
       app: App1
     template:
       metadata:
         labels:
           app: App1
           function: Front-end
       spec:
         containers:
         - name: simple-webapp
           image: simple-webapp

## Los valores de Labels ubicados en la parte superior del fichero yaml pertenecen a la replica, los de la parte inferior pertenece al pod

## Después se puede enlazar el fichero de replicas con uno de servicios:
nano service.yaml
apiVersion: v1
kind: Service
metadata:
    name: my-service
spec:
  selector:
    app: App1
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376

## 28.4º Las anotaciones son comentarios que permiten guardar el atributo buildversion, se coloca con annotations:
nano fichero.yaml
(before the spec attribute)
annotations:
    buildversion: 1.35

## 28.5º Con este comando se puede filtrar los pods según la aplicación en curso:
kubectl get pods --selector=bu
NAME          READY   STATUS    RESTARTS   AGE
app-1-8p8f9   1/1     Running   0          2m
app-1-blrn8   1/1     Running   0          2m
app-1-ncpdj   1/1     Running   0          2m
app-1-zzxdf   1/1     Running   0          2m
auth          1/1     Running   0          2m
db-2-ln6cd    1/1     Running   0          2m

## 29º Para filtrar el acceso de los pods al nodo establecemos el filtro taint:
kubectl tain nodes <node_name> key=value:taint-effect
## el taint-effect evaluará si el pod tolera el filtro
## 1º Filtro --> NoSchedule: el pod no recibe el shedule cuando vaya a crearse
## 2º Filtro --> PreferNoSchedule: el pod no tiene garantizado que se le aplique el NoSchedule
## 3º Filtro --> NoExecute: los pods serán desalojados del nodo que cumple con la tolerancia

## Muestra de un comando válido:
kubectl taint nodes node1 app=myapp:NoSchedule

## Comapración entre el comando y el fichero.yaml:
kubectl taint nodes node1 app*1=*2blue*3:NoSchedule*4

nano pod-tain-definition.yaml
apiVersion: v1
kind: Pod
metadata:
 name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
  tolerations:
  - key: app*1
    operator: "Equal"*2
    value: blue*3
    effect: NoSchedule*4

## Para mostrar si el taint está activo le pasamos el grep Taints
kubectl describe node <name_of_node> | grep Taint

## Muestra completa de fichero_taint completo:
cat bee_pod.yaml 
apiVersion: v1
kind: Pod
metadata:
 name: bee
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
  - key: spray
    operator: "Equal"
    value: mortein
    effect: NoSchedule

## Obtener los pods al completo:
kubectl get pods -o wide 
NAME       READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
bee        1/1     Running   0          2m55s   172.17.1.2   node01   <none>           <none>
mosquito   0/1     Pending   0          11m     <none>       <none>   <none>           <none>

## borrador de taint:
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
node/controlplane untainted

## 30º El tamaño del cluster se puede modificar dentro del fichero de construir_Pods.yaml

## Comando para asignar a los nodos el tamaño solicitado por el pod
kubectl label nodes <node_name> <label-key>=<label_value>
kubectl label nodes node-1 size=Large

## nano fichero_pod_size.yaml
apiVersion: v1
kind: Pod
metada:
 name: myapp-pod
spec:
  containers:
  - name: data-processor
    image: data-proccesor
  nodeSelector:
    size: Large

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
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   17m   v1.35.0   10.244.56.22    <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    <none>          17m   v1.35.0   10.244.34.242   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Estado de pods
kubectl get pods -o wide 
NAME                    READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
blue-56c45fd5ff-26hfq   1/1     Running   0          2m3s   172.17.1.2   node01         <none>           <none>
blue-56c45fd5ff-8b67j   1/1     Running   0          2m3s   172.17.1.3   node01         <none>           <none>
blue-56c45fd5ff-8zhnl   1/1     Running   0          2m3s   172.17.0.4   controlplane   <none>           <none>

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
                                       | BEST_OPTION
NO REQUESTS  | NO REQUESTS |  REQUESTS |    REQUESTS
NO LIMITS    |    LIMITS   |  LIMITS   | NO LIMITS
 ##  | #     |     |       |     |     | #    | #
 #   | #     |     |       |     |     | #    | #
 #   | #     |3vCPU| 3Gi   |3vCPU| 3Gi | #    | #
 #   | #     | # # | # #   | #   | #   | # #  | # #
 # # | # #   | # # | # #   |1vCPU| 1Gi |1vCPU | 1Gi
 1 2 | 1 2   | 1 2 | 1 2   | 1 2 | 1 2 | 1 2  | 1 2 
 CPU | RAM   | CPU | RAM   | CPU | RAM | CPU  | RAM
              REQUESTS=LIMITS
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

## Ambos ficheros se crean con kubectl create -f <file_name.yaml>
kubectl create -f replicaset-definition.yaml
kubectl create -f daemonset-definition.yaml

## Para consultar el daemonset usaremos:
kubectl get daemonsets

## Para verificar la estructura usaremos:
kubectl describe daemonset <daemonset_name>

## Mostrar todos los daemos activos
kubectl get daemonsets.apps --all-namespaces 
NAMESPACE      NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   kube-flannel-ds   1         1         1       1            1           <none>                   12m
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
static-greenbox-node01        1/1     Running   0              15s    172.17.1.3   node01         <none>           <none>

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
NAME              TYPE      NOMINALCONCURRENCYSHARES   QUEUES   HANDSIZE   QUEUELENGTHLIMIT   AGE
catch-all         Limited   5                          <none>   <none>     <none>             17m
exempt            Exempt    <none>                     <none>   <none>     <none>             17m
global-default    Limited   20                         128      6          50                 17m
leader-election   Limited   10                         16       4          50                 17m
node-high         Limited   40                         64       6          50                 17m
system            Limited   30                         64       6          50                 17m
workload-high     Limited   40                         128      6          50                 17m
workload-low      Limited   100                        128      6          50                 17m

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
Node:             <none>
Labels:           <none>
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Containers:
  critical-container:
    Image:      nginx
    Port:       <none>
    Host Port:  <none>
    Limits:
      memory:  3Gi
    Requests:
      cpu:        1
      memory:     3Gi
    Environment:  <none>
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
Node-Selectors:              <none>
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

## 38º El sistema de kubernetes permite expandirse y ofrecer más de dos planificadores para los pods, nodos, contenedores y personalizarlos. Cuando hay varios planificadores cada uno debe llevar distinto nombre para diferenciarse 