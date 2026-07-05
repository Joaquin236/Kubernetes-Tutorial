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

5º Pausar el minikube sin afectar a los servicios activos:
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

kubectl create -f replicaset.yaml

## 14.2º Consultar el set de réplicas activas
kubectl get replicaset

## 14.3º Borrar el set de réplicas activas, borrará todo el contenido del set
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
## El manifiesto del Deployment también debe realizarse manualmente
kubectl apply -f <name_file.yaml>

## 17º Este comando redirige la salida de la interfaz de consola a un fichero nuevo. Si existe sobreescribe su contenido
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
## En las versiones más modernas de Kubernetes admite personalizar el número de replicas a trabajar:
kubectl create deployment --image=nginx nginx --dry-run=client --replicas=4 -o yaml > nginx-deployment.yaml

## 18º Los servicios ofrecen interacción con los pods a través del comando curl <http://IP:puerto>
## Las direcciones IP de los pods no son enrutables con la dirección del host. Impidiendo acceder desde la navegación tradicional
## Dentro del nodo se encuentra un servicio que conecta el host con los contenedores activos
## Tipos de servicios: ["Node_Port" , "Cluster_IP" , "Load_Balancer"]
## Node_Port --> Cada contenedor lleva el puerto con su dirección con otra CIDR, el servicio hace de enlace entre el host y el nodo, este servicio tiene su dirección diferente a la del contenedor
## Fichero yaml del servicio NodePort
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
## Fichero yaml del pod asociado al servicio, los parámetros del labels se asocian con el yaml del servicio
apiVersion: v1
kind: Pod
metadata:
 name: myapp-pod
 labels:
    app: myapp
    type: front-end
## Comando para crear el servicio
kubectl creade -f <file_name.yaml>
## Comando para consultar los servicios
kubectl get services
## Después de configurarlo, el navegador GUI del cliente puede visitar el contenido del contenedor que queremos visitar
## Cuando hay más de un contenedor, el nodo reparte el servicio entre los contenedores. Cuando hay más de un cluster, el servicio se expande a los demás clusters
## En el segundo ejemplo de los node_ports, se ofrecen más de dos direcciones para enlazarse con los servidores. Si se produce un cambio, el servidor de nodos cambia la estructura sin necesidad de actuar

## 19º En el diseño del Cluster_IP los servidores de los contenedores se agrupan por sus funciones:
## Front-end --> Back-end --> Redis. Cada uno tiene la IP interna del nodo
## Fichero del Servicio ClusterIP
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

## Fichero del Pod asociado al servicio
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
## Fichero yaml del LoadBalancer
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
## Fichero NodePort_2
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
## También se pude establecer el namespace desde el fichero.yaml
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

## 22.2º Los namespaces admiten ser creados desde un fichero.yaml
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

## 25.4º Crear un deployment con 3 replicas y con imagen kodekloud
kubectl create deployment webapp --replicas=3 --image=kodekloud/webapp-color
deployment.apps/webapp created

## 25.5º Crear un pod con puerto 8080
kubectl run custom-nginx --image=nginx --port=8080
pod/custom-nginx created

## 25.6º Crear un namespace nuevo
kubectl create namespace dev-ns
namespace/dev-ns created

## 25.7º Crear un deploy con el enlace a un namespace
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

## 27.3º Cambiamos el nodo sin número por node02
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

## 27.5º Comando curl completo, este comando debe ser modificado para adaptarlo a la situación
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

## 28º 
