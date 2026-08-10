## 115.1º Introducción a los componentes
- Los componentes ofrecen la habilidad de definir piezas de configuración lógica, que pueden incluir en multiples overlays
- Los componentes son útiles en situaciones donde las aplicaciones soportan multiples opciones que necesitan activarse solo debajo de los overlays

## 115.2º En la aplicación que se quiere desplegar lleva esta estructura. Los directorios ["Premium","Self_Hosted"] pertenecen al Caching, mientras los directorios ["Dev","Premium"] pertenecen al External DB
/root/code/k8s/
└── raiz_de_app
    ├── base
    ├── dev
    ├── Premium
    └── Self_Hosted

## 115.3º Si se realizan cambios en este tipo de aplicaciones, necesitarás cambiar los demás componentes para que funcionen bien. 
└── raiz_de_app
    ├── base
    ├── dev         --> ["DB"]
    ├── Premium     --> ["Caching","DB"]
    └── Self_Hosted --> ["Caching"]

## 115.4º Esta estructura incorpora el subdirectorio de componentes donde se establece las funciones de la aplicación, también está el overlays, caca uno tiene un fichero de kustomización 
/root/code/k8s/
├── base/
|   ├── kustomization.yaml
|   └── api-depl.yaml
├── components/
|   ├── caching/
|   |   ├── kustomization.yaml
|   |   ├── deployment-patch.yaml
|   |   └── redis-depl.yaml
|   └── db/
|       ├── kustomization.yaml
|       ├── deployment-patch.yaml
|       └── postgres-depl.yaml
└── overlays
    ├── dev/
    |   └── kustomization.yaml
    ├── premium/
    |   └── kustomization.yaml
    └── standalone/
        └── kustomization.yaml

## 115.5º Este fichero crea un servicio de bases de datos con el motor postgres, se aplica con kubectl -k /root/code/k8s/components/db
nano /root/code/k8s/components/db/postgres-depl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment 
spec:
  replicas: 1
  selector:
    matchLabels:
      component: postgres 
  template:
    metadata:
      labels:
        component: postgres 
    spec:
      containers:
        - name: postgres
          image: postgres

## 115.6º Este fichero crea un secreto para proteger la contraseña de la base de datos, se aplica con kubectl -k /root/code/k8s/components/db
nano /root/code/k8s/components/db/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1apha1
kind: Component 
# Localización del fichero de recursos
resources:
  - postgres-depl.yaml
  
# Objeto secreto con la contraseña
secretGenerator:
  - name: postgres-cred 
    literals:
      - password=postgres123
      
# Localización del fichero para parchear
patches:
  - deployment-patch.yaml

## 115.7º Este fichero contiene el parche y el despliegue de un entorno de bases de datos postgres
nano /root/code/k8s/components/db/deployment-patch.yaml
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
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-cred
                  key password

## 115.8º Este fichero se encarga de la personalización de los directorios que está marcando como objetivos
nano /root/code/k8s/overlays/dev/kustomization.yaml
# Localizar y subir al directorio base
bases:
  - ../../base
# Localizar y subir al directorio components/db
  - ../../components/db
