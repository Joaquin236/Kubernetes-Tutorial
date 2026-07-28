## Consultar el ingress activo
kubectl -n webapp get svc
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
default-backend   ClusterIP   172.20.121.220   [none]        80/TCP    75s
web-app           ClusterIP   172.20.154.243   [none]        80/TCP    75s

## Consultar el secreto activo
kubectl -n webapp get secrets
NAME      TYPE                DATA   AGE
app-tls   kubernetes.io/tls   2      74s

## Consultar el deploy del ns ingress-nginx
kubectl -n ingress-nginx get deploy
NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
ingress-nginx-controller   1/1     1            1           74s

## Crea un fichero para desplegar un Ingress nuevo
nano ingress_nginx.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: web-app-ingress
 namespace: webapp
spec:
  ingressClassName: nginx
  rules:
  - host: app.kodekloud.local
    http:
     paths:
     - path: /
       pathType: Prefix
       backend:
        service:
         name: web-app
         port:
          number: 80
kubectl apply -f ingress_nginx.yaml 
ingress.networking.k8s.io/web-app-ingress created

## Edita el ingress para adaptarlo y añadir el tls
nano ingress_nginx.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: web-app-ingress
 namespace: webapp
spec:
  ingressClassName: nginx
  tls:
  - hosts:
     - app.kodekloud.local
    secretName: app-tls
  rules:
  - host: app.kodekloud.local
    http:
     paths:
     - path: /
       pathType: Prefix
       backend:
        service:
         name: web-app
         port:
          number: 80
kubectl apply -f ingress_nginx.yaml 
ingress.networking.k8s.io/web-app-ingress configured

## Vuelve a editar el fichero yaml para añadir una anotación
nano ingress_nginx.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: web-app-ingress
 namespace: webapp
 annotations:
  nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
     - app.kodekloud.local
    secretName: app-tls
  rules:
  - host: app.kodekloud.local
    http:
     paths:
     - path: /
       pathType: Prefix
       backend:
        service:
         name: web-app
         port:
          number: 80
kubectl apply -f ingress_nginx.yaml 
ingress.networking.k8s.io/web-app-ingress configured

## Consulta el estado del ingress en el ns webapp
kubectl -n webapp get ingress -o wide 
NAME              CLASS   HOSTS                 ADDRESS        PORTS     AGE
web-app-ingress   nginx   app.kodekloud.local   172.20.79.15   80, 443   14m

## Describe el ingress web-app-ingress
kubectl -n webapp describe ingress web-app-ingress
Name:             web-app-ingress
Labels:           [none]
Namespace:        webapp
Address:          172.20.79.15
Ingress Class:    nginx
Default backend:  [default]
TLS:
  app-tls terminates app.kodekloud.local
Rules:
  Host                 Path  Backends
  ----                 ----  --------
  app.kodekloud.local  
                       /   web-app:80 (172.17.0.4:80)
Annotations:           nginx.ingress.kubernetes.io/ssl-redirect: true
Events:
  Type    Reason  Age                  From                      Message
  ----    ------  ----                 ----                      -------
  Normal  Sync    3m24s (x4 over 13m)  nginx-ingress-controller  Scheduled for sync

## Consulta la URL para visualizar el mensaje de conexión exitosa
curl -Lk https://app.kodekloud.local
Welcome to the secure web app!
