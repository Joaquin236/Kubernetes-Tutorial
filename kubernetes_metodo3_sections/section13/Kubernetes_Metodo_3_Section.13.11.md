##
tree /root/
/root/
└── code
    ├── README.md
    └── k8s
        ├── base
        │   ├── api-deployment.yaml
        │   ├── db-configMap.yaml
        │   ├── kustomization.yaml
        │   └── mongo-depl.yaml
        └── overlays
            ├── QA
            │   └── kustomization.yaml
            ├── dev
            │   ├── api-patch.yaml
            │   └── kustomization.yaml
            ├── prod
            │   ├── api-patch.yaml
            │   ├── kustomization.yaml
            │   └── redis-depl.yaml
            └── staging
                ├── configMap-patch.yaml
                └── kustomization.yaml

8 directories, 13 files

##
grep image /root/code/k8s/overlays/prod/api-patch.yaml 
          image: memcached

##
grep replicas /root/code/k8s/overlays/prod/redis-depl.yaml 
  replicas: 2

##
grep super  /root/code/k8s/overlays/staging/configMap-patch.yaml 
  password: superp@ssword123

##
El despliegue lanza 2 contenedores de Memcached, 2 conetedores de Nginx y 1 de base de datos mongodb

##
grep -A10 env /root/code/k8s/overlays/dev/api-patch.yaml 
          env:
            - name: DB_USERNAME
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                configMapKeyRef:
                  name: db-creds
                  key: password

##
grep -A10 env /root/code/k8s/base/api-deployment.yaml 
          env:
            - name: DB_CONNECTION
              value: db.kodekloud.com

##
nano /root/code/k8s/overlays/QA/kustomization.yaml
resources:
  - ../../base
  
labels:
  - pairs:
      environment: QA
    includeSelectors: false

patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: caddy

##
kubectl apply -k /root/code/k8s/overlays/QA/
configmap/db-creds created
deployment.apps/api-deployment created
deployment.apps/mongo-deployment created

##
nano /root/code/k8s/overlays/staging/mysql-depl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: mysql
  template:
    metadata:
      labels:
        component: mysql
    spec:
      containers:
        - name: mysql
          image: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: mypassword

##
nano /root/code/k8s/overlays/staging/kustomization.yaml 
resources:
  - ../../base
  - mysql-depl.yaml

labels:
  - pairs:
      environment: staging
    includeSelectors: false

##
kubectl apply -k /root/code/k8s/overlays/staging/
configmap/db-creds unchanged
deployment.apps/api-deployment unchanged
deployment.apps/mongo-deployment unchanged
deployment.apps/mysql-deployment created
