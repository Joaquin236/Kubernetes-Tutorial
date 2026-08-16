## Crear el storage_class
nano local-sc.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
kubectl apply -f local-sc.yaml 
storageclass.storage.k8s.io/local-sc created
kubectl get storageclass / kubectl get storageclasses.storage.k8s.io 
NAME                 PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-sc (default)   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   true                   15s

## Crea un despliegue con los valores establecidos
nano logging-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: logging-deployment
  namespace: logging-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: logger
  template:
    metadata:
      labels:
        app: logger
    spec:
      volumes:
        - name: log-volume
          emptyDir: {}
      initContainers:
        - name: log-agent
          image: busybox
          command:
            - sh
            - -c
            - "touch /var/log/app/app.log; tail -f /var/log/app/app.log"
          volumeMounts:
            - name: log-volume
              mountPath: /var/log/app
          restartPolicy: Always 
      containers:
        - name: app-container
          image: busybox
          command:
            - sh
            - -c
            - "while true; do echo 'Log entry' >> /var/log/app/app.log; sleep 5; done"
          volumeMounts:
            - name: log-volume
              mountPath: /var/log/app

## Crear el ns si aún no se ha creado
kubectl create namespace logging-ns
Error from server (AlreadyExists): namespaces "logging-ns" already exists

## Cambiar el ns a logging-ns
kubectl config set-context --current --namespace logging-ns 
Context "kubernetes-admin@kubernetes" modified.

## Crea el directorio y crea el fichero para el registro
mkdir -vp /var/log/app ; touch /var/log/app/app.log
mkdir: created directory '/var/log/app'

## Aplica el deploy
kubectl apply -f logging-deploy.yaml 
deployment.apps/logging-deployment created

## Verificar el registro de agente
kubectl logs -n logging-ns deployment/logging-deployment -c log-agent
Log entry
Log entry
Log entry
Log entry
Log entry
Log entry

## Cambiar a ingress-ns
kubectl config set-context --current --namespace ingress-ns

## Verifica los registros del deploy en ingress-ns
kubectl get deployment webapp-deploy -n ingress-ns
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
webapp-deploy   1/1     1            1           83s

kubectl get svc webapp-svc -n ingress-ns
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
webapp-svc   ClusterIP   172.20.217.156   <none>        80/TCP    105s

## Crea un ingress
nano webapp-svc.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  namespace: ingress-ns
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: kodekloud-ingress.app
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-svc
            port:
              number: 80
kubectl apply -f webapp-svc.yaml
ingress.networking.k8s.io/webapp-ingress created

## Visita de la página web con CLI-CUI
curl -s http://kodekloud-ingress.app/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

## Regresar al default
kubectl config set-context --current --namespace default

## Crea un deploy actualizable con nginx
kubectl create deployment nginx-deploy --image=nginx:1.16 --dry-run=client -o yaml > deploy.yaml
kubectl apply -f deploy.yaml --record
kubectl rollout history deployment nginx-deploy
kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
kubectl rollout history deployment nginx-deploy

## Consulta el pod
kubectl get pods -o wide 
NAME                            READY   STATUS    RESTARTS   AGE   IP            NODE     NOMINATED NODE   READINESS GATES
nginx-deploy-5f98968576-mr94j   1/1     Running   0          44s   172.17.1.13   node01   <none>           <none>


## Crea un usuario y autoriza sus labores de administración
adduser john
nano crs_manifest_john.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john-developer
spec:
  signerName: kubernetes.io/kube-apiserver-client
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZEQ0NBVHdDQVFBd0R6RU5NQXNHQTFVRUF3d0VhbTlvYmpDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUt2Um1tQ0h2ZjBrTHNldlF3aWVKSzcrVVdRck04ZGtkdzkyYUJTdG1uUVNhMGFPCjV3c3cwbVZyNkNjcEJFRmVreHk5NUVydkgyTHhqQTNiSHVsTVVub2ZkUU9rbjYra1NNY2o3TzdWYlBld2k2OEIKa3JoM2prRFNuZGFvV1NPWXBKOFg1WUZ5c2ZvNUpxby82YU92czFGcEc3bm5SMG1JYWpySTlNVVFEdTVncGw4bgpjakY0TG4vQ3NEb3o3QXNadEgwcVpwc0dXYVpURTBKOWNrQmswZWhiV2tMeDJUK3pEYzlmaDVIMjZsSE4zbHM4CktiSlRuSnY3WDFsNndCeTN5WUFUSXRNclpUR28wZ2c1QS9uREZ4SXdHcXNlMTdLZDRaa1k3RDJIZ3R4UytkMEMKMTNBeHNVdzQyWVZ6ZzhkYXJzVGRMZzcxQ2NaanRxdS9YSmlyQmxVQ0F3RUFBYUFBTUEwR0NTcUdTSWIzRFFFQgpDd1VBQTRJQkFRQ1VKTnNMelBKczB2czlGTTVpUzJ0akMyaVYvdXptcmwxTGNUTStsbXpSODNsS09uL0NoMTZlClNLNHplRlFtbGF0c0hCOGZBU2ZhQnRaOUJ2UnVlMUZnbHk1b2VuTk5LaW9FMnc3TUx1a0oyODBWRWFxUjN2SSsKNzRiNnduNkhYclJsYVhaM25VMTFQVTlsT3RBSGxQeDNYVWpCVk5QaGhlUlBmR3p3TTRselZuQW5mNm96bEtxSgpvT3RORStlZ2FYWDdvc3BvZmdWZWVqc25Yd0RjZ05pSFFTbDgzSkljUCtjOVBHMDJtNyt0NmpJU3VoRllTVjZtCmlqblNucHBKZWhFUGxPMkFNcmJzU0VpaFB1N294Wm9iZDFtdWF4bWtVa0NoSzZLeGV0RjVEdWhRMi80NEMvSDIKOWk1bnpMMlRST3RndGRJZjAveUF5N05COHlOY3FPR0QKLS0tLS1FTkQgQ0VSVElGSUNBVEUgUkVRVUVTVC0tLS0tCg==
  usages:
  - digital signature
  - key encipherment
  - client auth
kubectl apply -f crs_manifest_john.yaml
certificatesigningrequest.certificates.k8s.io/john-developer created

## Autorizaciones para el ususario
kubectl certificate approve john-developer ; kubectl create role developer --resource=pods --verb=create,list,get,update,delete --namespace=development ; kubectl create rolebinding developer-role-binding --role=developer --user=john --namespace=development ; kubectl auth can-i update pods --as=john --namespace=development
certificatesigningrequest.certificates.k8s.io/john-developer approved
role.rbac.authorization.k8s.io/developer created
rolebinding.rbac.authorization.k8s.io/developer-role-binding created


## Inicia el pod nginx-resolver
kubectl run nginx-resolver --image=nginx
kubectl expose pod nginx-resolver --name=nginx-resolver-service --port=80 --target-port=80 --type=ClusterIP
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup nginx-resolver-service > /root/CKA/nginx.svc
kubectl get pod nginx-resolver -o wide
kubectl run test-nslookup --image=busybox:1.28 --rm -it --restart=Never -- nslookup <P-O-D-I-P.default.pod> > /root/CKA/nginx.pod

## Crea un pod para el nodo node01
kubectl run nginx-critical --image=nginx --dry-run=client -o yaml > static.yaml
scp static.yaml node01:/root/
kubectl get nodes -o wide

# Perform SSH
ssh node01

mkdir -vp /etc/kubernetes/manifests
cp -vr /root/static.yaml /etc/kubernetes/manifests/
exit
logout
kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
nginx-critical-node01           1/1     Running   0          19s
nginx-deploy-5f98968576-mr94j   1/1     Running   0          7m18s
nginx-resolver                  1/1     Running   0          3m49s

## Crea un horizontal scaler
nano backend-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment
  minReplicas: 3
  maxReplicas: 15
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 65
kubectl apply -f backend-hpa.yaml
horizontalpodautoscaler.autoscaling/backend-hpa created

## Edita el gateway existente en el ns cha5673
kubectl config set-context --current --namespace cka5673 
Context "kubernetes-admin@kubernetes" modified.

kubectl get gateway
NAME          CLASS       ADDRESS   PROGRAMMED   AGE
web-gateway   kodekloud             Unknown      3m26s

kubectl get gateway web-gateway -n cka5673 -o yaml
# plantilla inicial
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: cka5673
  resourceVersion: "21919"
  uid: a1d0e35d-5126-4000-88ec-f440941eed75
spec:
  gatewayClassName: kodekloud
  listeners:
  - allowedRoutes:
      namespaces:
        from: Same
    name: https
    port: 80
    protocol: HTTP

## El nuevo gateway debe ser así
nano web-gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: cka5673
spec:
  gatewayClassName: kodekloud
  listeners:
    - name: https
      protocol: HTTPS
      port: 443
      hostname: kodekloud.com
      tls:
        certificateRefs:
          - name: kodekloud-tls
kubectl apply -f web-gateway.yaml
gateway.gateway.networking.k8s.io/web-gateway configured

## Con la herramienta helm, borra un chart vulnerable
helm ls -A
helm ls -A
NAME                    NAMESPACE               REVISION        UPDATED                                 STATUS          CHART                          APP VERSION
atlanta-page-apd        atlanta-page-04         1               2026-08-16 21:42:22.424135992 +0000 UTC deployed        atlanta-page-apd-0.1.0         1.16.0     
digi-locker-apd         digi-locker-02          1               2026-08-16 21:42:21.81294017 +0000 UTC  deployed        digi-locker-apd-0.1.0          1.16.0     
security-alpha-apd      security-alpha-01       1               2026-08-16 21:42:21.535707489 +0000 UTC deployed        security-alpha-apd-0.1.0       1.16.0     
web-dashboard-apd       web-dashboard-03        1               2026-08-16 21:42:22.114801552 +0000 UTC deployed        web-dashboard-apd-0.1.0        1.16.0     

kubectl get deploy -n <NAMESPACE> <DEPLOYMENT-NAME> -o json | jq -r '.spec.template.spec.containers[].image'

kubectl get deploy -n atlanta-page-04 atlanta-page-apd  -o json | jq -r '.spec.template.spec.containers[].image'
kodekloud/webapp-color:v1

helm uninstall <RELEASE-NAME> -n <NAMESPACE>

helm uninstall atlanta-page-apd-0.1.0 atlanta-page-04 --> Error: uninstall: Release not loaded: atlanta-page-apd-0.1.0: release: not found
helm uninstall digi-locker-apd-0.1.0 digi-locker-02 --> Error: uninstall: Release not loaded: digi-locker-apd-0.1.0: release: not found
helm uninstall security-alpha-apd-0.1.0  security-alpha-01 --> Error: uninstall: Release not loaded: security-alpha-apd-0.1.0: release: not found
helm uninstall web-dashboard-apd-0.1.0  web-dashboard-03 --> Error: uninstall: Release not loaded: web-dashboard-apd-0.1.0: release: not found


## de los 3 net-pol ofrecidos solo hay uno válido, instala el que es válido
cat /root/net-pol-1.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: net-policy-1
  namespace: backend
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: allowed
    ports:
    - protocol: TCP
      port: 80

cat /root/net-pol-2.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: net-policy-2
  namespace: backend
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    - namespaceSelector:
        matchLabels:
          name: databases
    ports:
    - protocol: TCP
      port: 80

cat /root/net-pol-3.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: net-policy-3
  namespace: backend
spec:
  podSelector: {}
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 80

Understand the differences:
net-pol-1.yaml: Too broad; allows traffic from any namespace with a certain label.
net-pol-2.yaml: Incorrect; explicitly allows both frontend and databases.
net-pol-3.yaml: Correct; only allows traffic from the frontend namespace.

kubectl apply -f /root/net-pol-3.yaml

kubectl get netpol -n backend
NAME           POD-SELECTOR   AGE
net-policy-3   <none>         0s
