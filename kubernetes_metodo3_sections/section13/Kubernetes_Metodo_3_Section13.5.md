## El sistema tiene este contenido en el directorio de usuario Root. Debemos desplegar el servicio de aplicaciones alojado en la estructura
tree /root/code
.
├── README.md
└── k8s
    ├── db
    │   ├── db-config.yaml
    │   ├── db-depl.yaml
    │   └── db-service.yaml
    ├── kustomization.yaml
    ├── message-broker
    │   ├── rabbitmq-config.yaml
    │   ├── rabbitmq-depl.yaml
    │   └── rabbitmq-service.yaml
    └── nginx
        ├── nginx-depl.yaml
        └── nginx-service.yaml

4 directories, 10 files

## Contenido del fichero db-config.yaml
nano /root/code/k8s/db/db-config.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-credentials
data:
  username: "root"
  password: "example"

## Contenido del fichero db-depl.yaml
cat k8s/db/db-depl.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: db
  template:
    metadata:
      labels:
        component: db
    spec:
      containers:
        - name: mongo
          image: mongo
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: db-credentials
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: db-credentials
                  key: username

## 
cat k8s/db/db-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: db-service
spec:
  selector:
    component: db-deployment
  ports:
    - protocol: "TCP"
      port: 27017
      targetPort: 27017
  type: NodePort

  ##
  cat k8s/message-broker/rabbitmq-config.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-credentials
data:
  username: "redis"
  password: "password123"

##  
cat k8s/message-broker/rabbitmq-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: rabbit-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: redis
  ports:
    - port: 5672
      targetPort: 5672

##
cat k8s/message-broker/rabbitmq-depl.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rabbitmq-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      component: rabbitmq
  template:
    metadata:
      labels:
        component: rabbitmq
    spec:
      containers:
        - name: rabbitmq
          image: rabbitmq

##
cat k8s/nginx/nginx-depl.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      component: nginx
  template:
    metadata:
      labels:
        component: nginx
    spec:
      containers:
        - name: nginx
          image: nginx

##
 cat k8s/nginx/nginx-service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    component: nginx
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 80
  type: NodePort

##
cat k8s/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
# kubernetes resources to be managed by kustomize
resources:
  - db/db-depl.yaml
  - db/db-service.yaml
  - db/db-config.yaml
  - message-broker/rabbitmq-config.yaml
  - message-broker/rabbitmq-depl.yaml
  - message-broker/rabbitmq-service.yaml
  - nginx/nginx-depl.yaml
  - nginx/nginx-service.yaml

##
kustomize build k8s/ | kubectl apply -f -
configmap/db-credentials created
configmap/redis-credentials created
service/db-service created
service/nginx-service created
service/rabbit-cluster-ip-service created
deployment.apps/db-deployment created
deployment.apps/nginx-deployment created
deployment.apps/rabbitmq-deployment created

##
kubectl get pods -o wide 
NAME                                   READY   STATUS    RESTARTS   AGE    IP           NODE           NOMINATED NODE   READINESS GATES
db-deployment-6657f99d45-v4n9h         1/1     Running   0          115s   172.17.0.6   controlplane   <none>           <none>
nginx-deployment-57d574b95-267hd       1/1     Running   0          115s   172.17.0.8   controlplane   <none>           <none>
nginx-deployment-57d574b95-5t44h       1/1     Running   0          115s   172.17.0.4   controlplane   <none>           <none>
nginx-deployment-57d574b95-7l4bg       1/1     Running   0          115s   172.17.0.5   controlplane   <none>           <none>
rabbitmq-deployment-7d9fb68c75-hb68j   1/1     Running   0          115s   172.17.0.7   controlplane   <none>           <none>

##
cat k8s/message-broker/rabbitmq-service.yaml | grep type
  type: ClusterIP

##
nano /root/code/k8s/db/kustomization.yaml
nano /root/code/k8s/message-broker/kustomization.yaml
nano /root/code/k8s/nginx/kustomization.yaml

##
nano /root/code/k8s/db/kustomization.yaml
resources:
  - db-depl.yaml
  - db-service.yaml
  - db-config.yaml

##
nano /root/code/k8s/message-broker/kustomization.yaml
resources:
  - rabbitmq-config.yaml
  - rabbitmq-depl.yaml
  - rabbitmq-service.yaml

##
nano /root/code/k8s/nginx/kustomization.yaml
resources:
  - nginx-depl.yaml
  - nginx-service.yaml

##
nano /root/code/k8s/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# kubernetes resources to be managed by kustomize
resources:
  - db/
  - message-broker/
  - nginx/
#Customizations that need to be made

##
tree /root/
/root/
└── code
    ├── README.md
    └── k8s
        ├── db
        │   ├── db-config.yaml
        │   ├── db-depl.yaml
        │   ├── db-service.yaml
        │   └── kustomization.yaml
        ├── kustomization.yaml
        ├── message-broker
        │   ├── kustomization.yaml
        │   ├── rabbitmq-config.yaml
        │   ├── rabbitmq-depl.yaml
        │   └── rabbitmq-service.yaml
        └── nginx
            ├── kustomization.yaml
            ├── nginx-depl.yaml
            └── nginx-service.yaml

5 directories, 13 files

##
kustomize build k8s/ | kubectl apply -f -
configmap/db-credentials created
configmap/redis-credentials created
service/db-service created
service/nginx-service created
service/rabbit-cluster-ip-service created
deployment.apps/db-deployment created
deployment.apps/nginx-deployment created
deployment.apps/rabbitmq-deployment created

##
kubectl get pods -o wide 
NAME                                   READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
db-deployment-6657f99d45-dhkrh         1/1     Running   0          62s   172.17.0.9    controlplane   <none>           <none>
nginx-deployment-57d574b95-7rl2v       1/1     Running   0          62s   172.17.0.11   controlplane   <none>           <none>
nginx-deployment-57d574b95-8t2pd       1/1     Running   0          62s   172.17.0.10   controlplane   <none>           <none>
nginx-deployment-57d574b95-pkr9s       1/1     Running   0          62s   172.17.0.12   controlplane   <none>           <none>
rabbitmq-deployment-7d9fb68c75-4dr8b   1/1     Running   0          61s   172.17.0.13   controlplane   <none>           <none>
rabbitmq-deployment-7d9fb68c75-cdqhk   1/1     Running   0          61s   172.17.0.14   controlplane   <none>           <none>
