## Listar el directorio del proyecto a procesar
controlplane ~ ➜  tree /root/
/root/
└── code
    ├── README.md
    └── project_mercury
        ├── base
        │   ├── api-depl.yaml
        │   ├── api-service.yaml
        │   └── kustomization.yaml
        ├── components
        │   ├── auth
        │   │   ├── api-patch.yaml
        │   │   ├── keycloak-depl.yaml
        │   │   ├── keycloak-service.yaml
        │   │   └── kustomization.yaml
        │   ├── db
        │   │   ├── api-patch.yaml
        │   │   ├── db-deployment.yaml
        │   │   ├── db-service.yaml
        │   │   └── kustomization.yaml
        │   └── logging
        │       ├── kustomization.yaml
        │       ├── prometheus-depl.yaml
        │       └── prometheus-service.yaml
        └── overlays
            ├── community
            │   └── kustomization.yaml
            ├── dev
            │   └── kustomization.yaml
            └── enterprise
                └── kustomization.yaml

11 directories, 18 files

## Localizar el fichero de customizar el overlays/community y localizar el componente que procesa
cat /root/code/project_mercury/overlays/community/kustomization.yaml 
bases:
  - ../../base

components:
  - ../../components/auth

## Localizar el fichero de customizar el overlays/dev y localizar los componentes que procesa
cat /root/code/project_mercury/overlays/dev/kustomization.yaml 
bases:
  - ../../base

components:
  - ../../components/auth
  - ../../components/db
  - ../../components/logging

## Localizar las variables de entorno del fichero de parches de bases de datos
grep -A8 env /root/code/project_mercury/components/db/api-patch.yaml 
          env:
            - name: DB_CONNECTION
              value: postgres-service
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: db-creds
                  key: password

## Localizar el fichero de customizar el overlays/db y localizar el secreto que genera al usarlo
cat /root/code/project_mercury/components/db/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - db-deployment.yaml
  - db-service.yaml

secretGenerator:
  - name: db-creds
    literals:
      - password=password1

patches:
  - path: api-patch.yaml

## Edita el fichero de customizar el overlays/community para añadir las bases y los componentes
nano /root/code/project_mercury/overlays/community/kustomization.yaml 
bases:
  - ../../base

components:
  - ../../components/auth
  - ../../components/logging

## Guarda y aplica los cambios para generar el cluster nuevo
kubectl apply -k /root/code/project_mercury/overlays/community
# Warning: 'bases' is deprecated. Please use 'resources' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
service/api-service created
service/keycloak-service created
service/prometheus-service created
deployment.apps/api-deployment created
deployment.apps/keycloak-deployment created
deployment.apps/prometheus-deployment created

## Crea el fichero de customizar el componente de caching y añade las rutas de los ficheros adyacentes
nano /root/code/project_mercury/components/caching/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component
resources:
 - redis-depl.yaml
 - redis-service.yaml

## Guarda y aplica los cambios
kubectl apply -k /root/code/
project_mercury/components/caching/
service/redis-service created
deployment.apps/redis-deployment created

## Crea el fichero de aplicar los parches dentro del directorio caching
nano /root/code/project_mercury/components/caching/api-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: api
          env:
            - name: REDIS_CONNECTION
              value: redis-service

## Crea el fichero de customizar el caching y añade las rutas necesarias
nano /root/code/project_mercury/components/caching/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component
resources:
 - redis-depl.yaml
 - redis-service.yaml

patches:
  - path: api-patch.yaml

## Aplica los cambios
kubectl apply -k /root/code/project_mercury/components/caching/

## Edita el fichero de customizar el directorio enterprise y añade las rutas ["auth","db","caching"]
nano /root/code/project_mercury/overlays/enterprise/kustomization.yaml
bases:
  - ../../base

components:
  - ../../components/auth
  - ../../components/db
  - ../../components/caching

## Aplica los cambios para re-generar el cluster y desplegar nuevos servicios
kubectl apply -k /root/code/project_mercury/overlays/enterprise/
# Warning: 'bases' is deprecated. Please use 'resources' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
secret/db-creds-dd6525th4g created
service/api-service unchanged
service/keycloak-service unchanged
service/postgres-service created
service/redis-service unchanged
deployment.apps/api-deployment configured
deployment.apps/keycloak-deployment unchanged
deployment.apps/postgres-deployment created
deployment.apps/redis-deployment unchanged

