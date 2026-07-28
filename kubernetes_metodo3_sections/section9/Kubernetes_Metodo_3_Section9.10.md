## 89.1º Ingress ofrece a los usuarios el acceso a su aplicación a través de una sola URL de acceso externo configurable para dirigir el tráfico a diferentes servicios dentro del cluster, impantando la seguridad SSL. 

## 89.2º Si imaginamos el Ingress como un equilibrador de carga siete integrado en el cluster usando los objetos creados en Kubernetes. El Ingress hay que establecerlo con el puerto de nodo o con un equilibrador de carga de la nube.

## 89.3º Empezamos con un proxy inverso o equilibrador de carga con nginx, traefik o haproxy. Kubernetes los configura para crear las tablas de enrutamiento de tráfico a los servicios. Durante la configuracion se definen rutas URL y el SSL. El Ingress se implementa como una solución compatible y se establece las reglas de configuración. Los recursos se definen con ficheros de despliegue similares a los de los objetos. 

## 89.4º Este fichero es el que necesitamos para desplegar el deploy con el Ingress
nano nginx-ingress-controller_deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
      name: nginx-ingress
    spec:
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
          env:
            - name: POD_name
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metada.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
kubectl apply -f nginx-ingress-controller_deploy.yaml

## 89.5º En la sección de los puertos se declara el acceso por puerto 80 y 443
ports:
  - name: http
    containterPort: 80
  - name: https
    containerPort: 443

## 89.6º A parte del deploy, desplegamos este servicio con estos valores
nano nginx-ingress_service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress
kubectl apply -f nginx-ingress_service.yaml

## 89.7º Necesitaremos desplegar esta cuenta de servicio para establecerla en el despliegue principal
nano nginx-ingress-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
kubectl apply -f nginx-ingress-serviceaccount.yaml

## 89.8º Un objeto intermedio entre las aplicaciones, el servidor, el wear y el ingress es el servicio del wear. Se declara con un fichero y se ejecuta con Backend
nano ingress-wear_file.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  defaultBackend:
    service:
      name: wear-service
      port: 80
kubectl apply -f ingress-wear_file.yaml

## 89.9º Los objetos creados se pueden consultar con
kubectl get ingress
kubectl get deployment
kubectl get configMap
kubectl get service
kubectl get serviceAccount

## 89.10º Cada una de estas reglas definen el uso de los nombres de dominio, desde canalizar el trafico hasta gestionar el contenido multimedia, administrar el stock, pago y la lista del carro virtual de la tienda alojada en el servidor. La cuarta regla se encarga de los detalles no mencionados en la descipcion. Distribucion del las direcciones del servidor web y las posibles reglas de conexion establecidas, se le conoce como ingress-rules
|URL  |www.my-online-store.com|www.wear.my-online-store.com|www.watch.my-online-store.com|Others_URLs|
|-----|-----------------------|----------------------------|-----------------------------|-----------|
|Rules|Rule_1                 |Rule_2                      |Rule_3                       |Rule_4     |

## Dentro de cada regla se establece los parametros necesarios y la url de acceso

## Regla-1 ['http://my-online-store.com/wear','http://www.my-online-store.com/watch','http://www.my-online-store.com/listen'] Tambien se necesita una URL para mostrar el mensaje de contenido no esta disponible

## Regla-2 ['www.wear.my-online-store.com/','www.wear.my-online-store.com/returns','www.wear.my-online-store.com/support'] Estas URLs ofrecen accesos a la tienda, facilitar la devolucion de compras, ofrecer soporte y atencion al cliente

## Regla-3 ['www.watch.my-online-store.com/','www.watch.my-online-store.com/movies','www.watch.my-online-store.com/TV'] Contienen el acceso a los datos multimedia y television en abierto desde internet

## Regla-4 ['http://listen.my-online-store.com/','http://eat.www.my-online-store.com/','http://drink.www.my-online-store.com/'] Almacena y administra el contenido que no se ha insertado en las reglas anteriores

## 89.11º La regla afecta a www.my-online-store.com, hay dos rutas que ofrecen contenido ['/wear','/watch'], el servidor esta conectado a las rutas y da acceso a los dos contenidos
nano ingress-wear.yaml
apiVerion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear
spec:
  defaultBackend:
    service:
      name: wear-service
      port: 80
kubectl create -f ingress-wear.yaml

nano ingress-wear-watch_1.yaml
apiVerion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          service:
            name: wear-service
            port: 80
      - path: /watch
        backend:
          service:
            name: watch-service
            port: 80
kubectl create -f ingress-wear-watch_1.yaml

## 89.12º Este fichero incorpora el parámetro del host
nano ingress-wear-watch_2.yaml
apiVerion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  - host: wear.my-online-storage.com
    http:
      paths:
      - path: /wear
        backend:
          service:
            name: wear-service
            port: 80
  - host: watch.my-online-store.com
    http:
      paths:
      - path: /watch
        backend:
          service:
            name: watch-service
            port: 80
kubectl create -f ingress-wear-watch_2.yaml
