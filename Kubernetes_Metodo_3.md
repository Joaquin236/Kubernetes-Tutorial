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
kubectl create configmap <config-name> \
--from-literal=<key>=<value>
##
kubectl create configmap app-config \
--from-literal=APP_COLOR=blue \
--from-literal=APP_MODE=prod
##
kubectl create configmap <config-name> \
--from-file=</path/to/file.yaml>
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

## 48.1º Los ficheros que estamos usando están en texto plano y pueden ser consultados por cualquiera, al suceder esto en caso de que puede ser fitrado datos sensibles, habrá problemas con la confidencialidad, integridad de credenciales y habrá que modificar las credenciales de urgencia. Con el sistema de los secretos codificamos los datos sensibles, también se puede migrar los datos al configmap pero no es el fichero apropiado para las contraseñas. Los secretos también tienen dos formas de ser creados: ["Comandos","Ficheros"]

## 48.2º Los valores que necesitamos codificar, encriptar o esconder digitalmente suelen ser:
["DB_HOST","mysql"]       # --> Host del sistema de la DB
["DB_User","root"]        # --> Usuario de la DB
["DB_password","passwd"]  # --> Contraseña de la DB

## Muestra de secreto creado por comando:
kubectl create secret generic <secret-name> \
--from-literal=<key>=<value>  \
--from-literal=<key>=<value> \
--from-literal=<key>=<value>
#
kubectl create secret generic app_secret \
--from-literal=DB_Host=mysql \
--from-literal=DB_User=root \
--from-literal=DB_Passwd=passwd

## Si se añade muchos campos se necesitará sustituir las entradas --from-literal=<key>=<value> se cambia por un fichero:
kubectl create secret generic app_secret \
--from-file=</path/to/file.yaml>

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
kubectl create -f <secret-data.yaml>

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
kubectl get secrets <name_secret> -o yaml

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
Labels:       <none>
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
controlplane   Ready    control-plane   5m28s   v1.35.0   10.244.170.133   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    <none>          4m56s   v1.35.0   10.244.147.128   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Obtener los deploys activos:
kubectl get deployments -o wide 
NAME   READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
blue   3/3     3            3           65s   nginx        nginx:alpine   app=blue

## Obtener los pods activos:
kubectl get pods -o wide 
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6466bf85df-d6wf7   1/1     Running   0          2m12s   172.17.1.3   node01         <none>           <none>
blue-6466bf85df-k57ll   1/1     Running   0          2m12s   172.17.0.4   controlplane   <none>           <none>
blue-6466bf85df-t4t5q   1/1     Running   0          2m12s   172.17.1.2   node01         <none>           <none>

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
controlplane   Ready                      control-plane   13m   v1.35.0   10.244.170.133   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready,SchedulingDisabled   <none>          12m   v1.35.0   10.244.147.128   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Después del mantenimiento vuelve a activarlo:
kubectl uncordon node01 
node/node01 uncordoned

## El nodo no recibió ningún pod existenete:
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   15m   v1.35.0   10.244.170.133   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    <none>          14m   v1.35.0   10.244.147.128   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
##
kubectl get pods -o wide 
NAME                    READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6466bf85df-bbvcc   1/1     Running   0          3m27s   172.17.0.6   controlplane   <none>           <none>
blue-6466bf85df-frfnn   1/1     Running   0          3m27s   172.17.0.5   controlplane   <none>           <none>
blue-6466bf85df-k57ll   1/1     Running   0          9m38s   172.17.0.4   controlplane   <none>           <none>

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
blue-6466bf85df-bbvcc   1/1     Running   0          8m42s   172.17.0.6   controlplane   <none>           <none>
blue-6466bf85df-frfnn   1/1     Running   0          8m42s   172.17.0.5   controlplane   <none>           <none>
blue-6466bf85df-k57ll   1/1     Running   0          14m     172.17.0.4   controlplane   <none>           <none>
hr-app                  1/1     Running   0          2m19s   172.17.1.4   node01         <none>           <none>

## Acordona el nodo node01
kubectl cordon node01
node/node01 cordoned

## Verifica el estado de los pods:
kubectl get pods -o wide 
NAME                      READY   STATUS    RESTARTS   AGE     IP           NODE           NOMINATED NODE   READINESS GATES
blue-6466bf85df-bbvcc     1/1     Running   0          14m     172.17.0.6   controlplane   <none>           <none>
blue-6466bf85df-frfnn     1/1     Running   0          14m     172.17.0.5   controlplane   <none>           <none>
blue-6466bf85df-k57ll     1/1     Running   0          20m     172.17.0.4   controlplane   <none>           <none>
hr-app-759cb554bf-sqt2z   1/1     Running   0          4m10s   172.17.1.5   node01         <none>           <none>

## 55º Las versiones de los componentes de kubernetes son compatibles entre sí sin necesidad de tener el más reciente pero si necesita actualizar se puede realizar desde los comandos: apt update ; apt full-upgrade && kubeadm upgrate. A parte de elegir las versiones de los componentes, cluster, nodos, etc.

## Verificar la versión de los nodos:
kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   25m   v1.34.0   10.244.127.208   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26
node01         Ready    <none>          24m   v1.34.0   10.244.56.39     <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26

## Localiza los taints en los dos nodos a través de la descripción:
kubectl describe nodes node01 | grep -i taint
Taints:             <none>

kubectl describe nodes controlplane | grep -i taint
Taints:             <none>

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
controlplane   Ready,SchedulingDisabled   control-plane   60m   v1.35.6   10.244.127.208   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26
node01         Ready,SchedulingDisabled   <none>          60m   v1.34.0   10.244.56.39     <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26

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

## 57.1º Los servicios de kubernetes deben ser protegidos en caso de que entre el servidor y el cliente hay un atacante de hombre en medio (man in the middle ["Cracker"]), los métodos de protección suele ser a través de cifrar con certificado y claves.

## 57.2º La autenticación determina quíen puede obtener el acceso: ["Static_Token_File","Certificates","External_Auth_Providers-LDAP","Service_Accounts"].

## 57.3º La autorización implementa controles de acceso basados en roles, el usuario se asocia en grupos con permisos restrictivos: ["RBAC_Auth","Node_Auth","ABAC_Auth","Webhook_Mode"]

## 57.4º El certificado TLS se establece para blindar los servicios del kube-apiserver, el kube-proxy, kube-scheduler, kubelet, etcd_cluster and kube_Controller.

## 57.5º La politica de red determina como se conectará los pods a cada cluster y lo que se pueda realizar conectado.

## 57.6º El cluster está atendido por el administrador/operador encargado del despliegue y por el desarrollador que construye la aplicación, evalúa como se desarrollará las actualizaciones de la aplicación y corrige las incidencias de la aplicación. El cliente se conecta para recibir la información obtenida del contenedor que ofrece sus datos, los bots automatizan algunos ajustes de la infraestructura. Los nodos, deploy y pods están conectados y dependen de autorizaciones y autenticaciones para enlazarse.

## 57.7º Los comandos para crear y consultar cuentas son:
kubectl create serviceaccount sa1
kubectl get serviceaccount

## 57.8º Cada cuenta de acceso es administrada por el sistema de kubernetes en lugar del sistema operativo. Las peticiones son interpretadas por kubernetes_api-server.

## 57.9º El kube-apiserver se autentica con certificados, el token_file y servicios de cuenta.

## 57.10º El sistema de autenticación guarda un fichero.csv con la información de los usuarios y grupos. La contraseña se guarda con tokens de codificación.

## 57.11º No se recomienda guardar los usuarios, contraseñnas y grupos en ficheros de texto plano que pueden ser interceptados por entidades ajenas al servicio.

## 57.12º El volumen de montaje debe pasar por el kubeadm setup.

## 57.13º Configura una autorización basada en roles para los usuarios nuevos.

## 58.1º Si el cliente entra en una conexión no protegida por cifrado TLS cuando sea interceptado por el Cracker, el craker se llevará los datos en texto plano y tomará el control de la cuenta del cliente, una de las soluciones es; el contenido se cifra con un certificado y clave privada, el servidor recibe la clave pública, si la clave es interceptada el contenido puede ser descrifrado por el cracker.

## 58.2º El cifrado asimétrico SSH es útil cuando no se ofrece el servicio de autenticación por usuario/contraseña, se puede usar a través de los comandos:
ssh-keygen
cat ~/.ssh/autorized_keys
ssh -i id_ras user1@server1
openssl genrsa -out my-bank.key 1024
openssl rsa -in my-bank.key -pubout > mybank.pem

## 58.3º Si el atacante aún está interesado en capturar la cuenta del usuario puede recrear la página web con el servicio. Si el usuario no percibe que la página que está usando es una web falsa, continuará con el proceso de iniciar sesión, aunque esté el contenido cifrado, el atacante recibe las claves públicas para descubrir las credenciales y tomar el control de la cuenta, el cliente puede no ser consciente de la situación o cuando lo descubre la cuenta de Crakeada. El navegador también puede evaluar el estado del certificado y si no es seguro, da el aviso de servicio no seguro, cada certificado está firmado por una autoridad certificativa y también las empresas de seguridad informática evaluan los certficidados. Las claves públicas llevan la extenxión: ["*.crt","*.pem"]. Las claves privadas ["*.key","*-key.pem"]. Los certificados llevan ["*.crt"].

## 59.1º El administrador debe establecer una conexión segura entre los componentes del cluster, el servidor interno, el planificador y otros nodos. El servidor lleva su certificado con su clave privada, cada componente del servidor lleva un grupo de certificados y claves.

## 59.2º El cliente también lleva sus certificados y clave privada, establece una conexión segura con el proxy del sistema, el etcd server, kubelet, el controller manager y planificador.

## 59.3º Se puede admitir tener más de una entidad certificadora.

## 60.1º Para crear el TSL para usarlo en kubernetes necesitamos el comando:
openssl genrsa -out ca.key 2048 # --> Generar la clave
openssl req -new -key ca.key -sub "/CN=KUBERNETES-CA" -out ca.csr # --> Firmar el certificado de respuesta
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt # --> Firmar el certificado

## 60.2º El usuario admin tiene que realizar el proceso de generar la clave, después generará la firma de certificado de respuesta y el certificado final. Esta forma de identificarse es más segura que los usuarios y contraseñas. Cada componente lleva el suyo con el formato system_user:kubernetes_component_server. Si realizamos 'curl htts://kube-apiserver:6443/api/v1 --key admin.key --cert admin.crt --cacert ca.crt. La consola debe mostrar el fichero JSON del pod en ejecución.

## 60.3º Otra forma de realizar esta acción es creando un fichero yaml donde guardar los parámetros del certificado.
nano kube-config.yaml
apiVersion: v1  
clusters:
- cluster:
    certificate-autority: ca.crt
    server: https://kube-apiserver:6443
  name: kubernetes
kind: Config
users:
- name: kubernetes-admin
  user:
    client_certificate: admin.crt
    client_key: admin.key

## 60.4º El etcd_server es desplegado como un cluster desde múltiples servidores y entorno de alta disponibilidad. Necesita crear certificados adicionales. Con el comando 'cat /etc/kubernetes/manifest/etcd.yaml' 
mostramos los valores de los parámetros del sistema,
entre ellos hay valores para los certificados. Los clientes también necesita disponer de cerificados válidos para el acceso. Los nodos deben estar autenticados y certificados durante la conexión.

## 60.5º El apiserver de kubernetes se puede declarar el certificado con un fihcero openssl.cnf
nano openssl.cnf
[req]
req_extensions=v3_req
distinguished_name=req_distinguished_name
[v3_req]
basicConstrains=CA:FALSE
keyUsage=nonRepudiation,
subjectAltName=@alt_names
[alt_names]
DNS.1=kubernetes
DNS.2=kubernetes.default
DNS.3=kubernetes.default.svc
DNS.4=kubernetes.default.svc.cluster.local
IP.1=10.96.0.1
IP.2=172.17.0.87

## 60.6º El fichero kubelet-config.yaml contiene las rutas de los cetificados:
nano kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/kubelet-node01.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/kubelet-node01.key"

## 61.1º Cuando entramos en un nuevo equipo de desarrollo de kubernetes
# empezamos como administrador del cluster a trabajar.
# Si tuviesemos que desplegar un servicio completo desde cero,
# necesitamos realizar los pasos para crear los certificados y 
# autorizaciones de los grupo para cada usuario.
# Verificar los ficheros:
cat /etc/kubernetes/manifest/kube-apiserver.service
cat /etc/kubernetes/manifest/kube-apiserver.yaml
cat /etc/kubernetes/pki/apiserver.crt


## 61.2º Con el comando ssl y el fichero /etc/kubernetes/pki/apiserver.crt
# obtenemos más información sobre el certificado apiserver.crt:
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
# Después debe aparecer un yaml con datos del apiserver

## 61.3º Tabla de la estrucitura de los certificados de kubernetes:
+--------------------------------------------------+-------------------------------+---------------+----------------+------------+
| Rutas de certificado                             | Nombre del CN                 | Nombre del ALT| Organización   | Issuer     |
| /etc/kubernetes/pki/apiserver.crt                | kube-apiserver                | DNS & IP      |                | kubernetes |
| /etc/kubernetes/pki/apiserver.key                |                               |               |                |            |
| /etc/kubernetes/pki/ca.crt                       | kubernetes                    |               |                | kubernetes |
| /etc/kubernetes/pki/apiserver-kubelet-client.crt | kube-apiserver-kubelet-client |               | system:masters | kubernetes |
| /etc/kubernetes/pki/apiserver-kubelet-client.key |                               |               |                |            |
| /etc/kubernetes/pki/apiserver-etcd-client.crt    | kube-apiserver-kubelet-client |               | system:masters | self       |
| /etc/kubernetes/pki/apiserver-etcd-client.key    |                               |               |                |            |
| /etc/kubernetes/pki/etcd/ca.crt                  | kubernetes                    |               |                | kubernetes |
+--------------------------------------------------+-------------------------------+---------------+----------------+------------+

# 61.4º Kubernetes registra eventos de los procesos y fallos detectados,
# en estos eventos se encuentran las versiones del sistema, direcciones IP, 
# el acceso a los certificados, las autorizaciones y la autenticación. 
# Los comandos son:
journalctl -u etcd.service -l
kubectl logs etcd-master
kubectl logs
# Si el sistema kubernetes se desconecta y necesitamos el panel de eventos, 
# necesitamos usar los comandos de docker:
crictl ps -a
crictl logs <"Container_name">

## Localizar el certificado apiserver.crt del fichero kube-apiserver.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep apiserver.crt
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt

## Localizar el certificado apiserver-etcd-client.crt
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep client
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt

## Localizar la clave apiserver-kubelet-client.key del fichero kube-apiserver.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep kubelet
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key

## Localizar el certificado server.crt del fichero etcd.yaml
cat /etc/kubernetes/manifests/etcd.yaml | grep server
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --key-file=/etc/kubernetes/pki/etcd/server.key
