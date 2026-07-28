## Consultar los insgress en los ns
kubectl get ingress -o wide --all-namespaces 
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS          PORTS   AGE
app-space   ingress-wear-watch   <none>   *       172.20.229.130   80      52s

## Consultar todos los controladores en los ns
kubectl get controllerrevisions.apps --all-namespaces 
NAMESPACE      NAME                        CONTROLLER                       REVISION   AGE
kube-flannel   kube-flannel-ds-985c47586   daemonset.apps/kube-flannel-ds   1          12m
kube-system    kube-proxy-69494898cd       daemonset.apps/kube-proxy        1          12m

## Consultar todos los objetos de todos los ns. [Es_importante_usar_grep_para_filtrar]
kubectl get all --all-namespaces | grep controller
ingress-nginx   pod/ingress-nginx-controller-7677dff578-x25hs   1/1     Running     0          9m46s
kube-system     pod/kube-controller-manager-controlplane        1/1     Running     0          18m
ingress-nginx   service/ingress-nginx-controller             NodePort    172.20.229.130   <none>        80:30080/TCP,443:32103/TCP   9m46s
ingress-nginx   service/ingress-nginx-controller-admission   ClusterIP   172.20.180.89    <none>        443/TCP                      9m46s
ingress-nginx   deployment.apps/ingress-nginx-controller   1/1     1            1           9m46s
ingress-nginx   replicaset.apps/ingress-nginx-controller-7677dff578   1         1         1       9m46s

## Consultar los pods de todos los ns y filtrar por space
kubectl get pods -A -o wide | grep space
app-space       default-backend-68fd4d68f-9bpbr  1/1     Running     0          14m   172.17.0.6      controlplane   <none>           <none>
app-space       webapp-video-68cff9d6fc-x5htk    1/1     Running     0          14m   172.17.0.5      controlplane   <none>           <none>
app-space       webapp-wear-7759c9f9d4-ttnv6     1/1     Running     0          14m   172.17.0.4      controlplane   <none>           <none>

## Consultar el ingress de los ns
kubectl get ingress -o wide -A
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS          PORTS   AGE
app-space   ingress-wear-watch   <none>   *       172.20.229.130   80      16m

## Descibir todos los ingress
kubectl describe ingress -A
Name:             ingress-wear-watch
Labels:           <none>
Namespace:        app-space
Address:          172.20.229.130
Ingress Class:    <none>
Default backend:  <default>
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /wear    wear-service:8080 (172.17.0.4:8080)
              /watch   video-service:8080 (172.17.0.5:8080)
Annotations:  nginx.ingress.kubernetes.io/rewrite-target: /
              nginx.ingress.kubernetes.io/ssl-redirect: false
Events:
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    18m (x2 over 18m)  nginx-ingress-controller  Scheduled for sync

## Consultar el yaml del deploy y filtrar por [default-back]
kubectl get deployments.apps ingress-nginx-controller -n ingress-nginx -o yaml # --> yaml completo
kubectl get deployments.apps ingress-nginx-controller -n ingress-nginx -o yaml | grep default-back
        - --default-backend-service=app-space/default-backend-service

## Extraer el yaml del ingress y redirigir a un fichero 
kubectl get ingress -n app-space -o yaml > ingress-wear-watch_file.yaml

## Adapta el fichero a las nuevas necesidades
nano ingress-wear-watch_file.yaml 
apiVersion: v1
items:
- apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
    creationTimestamp: "2026-07-28T15:21:49Z"
    generation: 1
    name: ingress-wear-watch
    namespace: app-space
    resourceVersion: "1341"
    uid: 8c69475d-ba20-4e8a-b81c-d1ac92c931b6
  spec:
    rules:
    - http:
        paths:
        - backend:
            service:
              name: wear-service
              port:
                number: 8080
          path: /wear
          pathType: Prefix
        - backend:
            service:
              name: video-service
              port:
                number: 8080
          path: /stream
          pathType: Prefix
  status:
    loadBalancer:
      ingress:
      - ip: 172.20.229.130
kind: List
metadata:
  resourceVersion: ""
kubectl apply -f ingress-wear-watch_file.yaml 
Warning: resource ingresses/ingress-wear-watch is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
ingress.networking.k8s.io/ingress-wear-watch configured

## Describe el deploy del ns app-space
kubectl describe deployments.apps -n app-space 

## Crea un nuevo fichero para tener un ingress nuevo
nano ingress-wear-watch_file.yaml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port: 
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port: 
              number: 8080
        path: /stream
        pathType: Prefix
      - backend:
          service:
            name: food-service
            port: 
              number: 8080
        path: /eat
        pathType: Prefix
kubectl apply -f ingress-wear-watch_file.yaml 
ingress.networking.k8s.io/ingress-wear-watch configured

## Consulta los deploy de todos los ns y localiza el critical-space
kubectl get  deployments.apps -A | grep criti
critical-space   webapp-pay                 1/1     1            1           112s

## Crea el fichero ingres-pay
nano ingress-pay_file.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: critical-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
           name: pay-service
           port:
            number: 8282
kubectl apply -f ingress-pay_file.yaml 
ingress.networking.k8s.io/critical-ingress created
