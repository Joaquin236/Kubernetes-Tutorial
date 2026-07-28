## 89.1 Ingress ofrece a los usuarios el acceso a su aplicacion a traves de una sola URL de acceso externo configurable para dirigir el trafico a diferentes servicios dentro del cluster, impantando la seguridad SSL. 

## 89.2 Si imaginamos el Ingress como un equilibrador de carga siete integrado en el cluster usando los objetos creadon en Kubernetes. El Ingress hay que establecerlo con el puerto de nodo o con un equilibrador de carga de la nube.

## 89.3 Empezamos con un proxy inverso o equilibrador de carga con nginx, traefik o haproxy. Kubernetes los configura para crear las tablas de enrutamiento de trafico a los servicios. Durante la configuracion se definen rutas URL y el SSL. El Ingress se implementa como una solucion compatible y se establece las reglas de configuracion. Los recursos se definen con ficheros de despliegue similares a los de los objetos. 

## 89.4 Este fichero es el que necesitamos para desplegar el deploy con el Ingress
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
