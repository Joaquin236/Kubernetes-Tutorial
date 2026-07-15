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
# Your deployment is now available at ["EXTERNAL-IP"]:8080

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
example-ingress   nginx   *       ["your_ip_here"]   80      5m45s
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
