## Listar el subdirectorio de k8s y los ficheros del despliegue
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

## Localizar la imagen del api patch del directorio prod
grep image /root/code/k8s/overlays/prod/api-patch.yaml 
          image: memcached

## Localizar las número replicas que despliega el fichero redis en el directorio prod 
grep replicas /root/code/k8s/overlays/prod/redis-depl.yaml 
  replicas: 2

## Localiza la contraseña que usa el configmap del directorio stanging
grep super  /root/code/k8s/overlays/staging/configMap-patch.yaml 
  password: superp@ssword123

## ¿Cuantos pods lanzará el despliegue cuando se active?
El despliegue lanza 2 contenedores de Memcached, 2 conetedores de Nginx y 1 de base de datos mongodb

## Localizar las variables de entorno del fichero api-patch
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

## Localizar las variables de entorno del fichero api-deploy
grep -A10 env /root/code/k8s/base/api-deployment.yaml 
          env:
            - name: DB_CONNECTION
              value: db.kodekloud.com

## Editar el fichero kustomization del directorio QA para modificar la imagen del QA
nano /root/code/k8s/overlays/QA/kustomization.yaml
# Subir hasta el directorio base
resources:
  - ../../base
# Atributo y variables de entorno  
labels:
  - pairs:
      environment: QA
    includeSelectors: false
# Parche para procesar y modificar el despliegue
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/spec/containers/0/image
        value: caddy

## Aplicar los cambios al entorno de QA
kubectl apply -k /root/code/k8s/overlays/QA/
configmap/db-creds created
deployment.apps/api-deployment created
deployment.apps/mongo-deployment created

## Crea un fichero para ofrecer un servicio de bases de datos MySQL
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

## Edita el fichero de customización del directorio staging 
nano /root/code/k8s/overlays/staging/kustomization.yaml 
resources:
  - ../../base
  - mysql-depl.yaml

labels:
  - pairs:
      environment: staging
    includeSelectors: false

## Aplica los cambios realizados a staging
kubectl apply -k /root/code/k8s/overlays/staging/
configmap/db-creds unchanged
deployment.apps/api-deployment unchanged
deployment.apps/mongo-deployment unchanged
deployment.apps/mysql-deployment created
