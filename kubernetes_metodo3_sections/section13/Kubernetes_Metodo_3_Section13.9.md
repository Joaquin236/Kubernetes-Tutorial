## Listar el directorio de trabajo con los ficheros para procesar
tree /root/
/root/
└── code
    ├── README.md
    └── k8s
        ├── kustomization.yaml
        ├── mongo-depl.yaml
        ├── mongo-label-patch.yaml
        ├── mongo-service.yaml
        └── nginx-depl.yaml

2 directories, 6 files

## Localizar el número de réplicas en el fichero de customizar el cluster
grep -A2 replicas /root/code/k8s/kustomization.yaml 
        path: /spec/replicas
        value: 3

## Localizar el componente del fichero mongo-depl.yaml
grep component /root/code/k8s/mongo-depl.yaml 
      component: mongo
  template:
    metadata:

## Localizar el tipo de cluster y el valor de caracteristica del mongo-label-patch.yaml
grep -A5 cluster /root/code/k8s/mongo-label-patch.yaml 
  path: /spec/template/metadata/labels/cluster
  value: staging

- op: add
  path: /spec/template/metadata/labels/feature
  value: db

## Localizar el puerto donde se ejecutará el sistema de bases de datos
grep -A3 port /root/code/k8s/kustomization.yaml 
        path: /spec/ports/0/port
        value: 30000

      - op: replace
        path: /spec/ports/0/targetPort
        value: 30000

## Localizar el contenedor del api-depl.yaml
 grep -A2 containers /root/code/k8s/api-depl.yaml 
      containers:
        - name: nginx
          image: nginx

## Localizar el contenedor del api-patch.yaml
grep -A2 containers /root/code/k8s/api-patch.yaml 
      containers:
        - name: memcached
          image: memcached

## Localizar la ruta de montaje del mongo-patch.yaml
grep mount /root/code/k8s/mongo-patch.yaml 
            - mountPath: /data/db

## Editar el fichero de customización para borrar la imagen de memcached
nano /root/code/k8s/kustomization.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
 template:
   spec:
    containers:
      - $patch: delete
        name: memcached

## Aplicar el nuevo sistema
kubectl apply -f /root/code/k8s/
deployment.apps/api-deployment unchanged
deployment.apps/mongo-deployment unchanged
service/mongo-cluster-ip-service unchanged

## Edita el fichero para realizar un borrado con Json6902 Inline y borrar el org del sistema
nano /root/code/k8s/kustomization.yaml
resources:
  - mongo-depl.yaml
  - api-depl.yaml
  - mongo-service.yaml

patches:
  - target:
      kind: Deployment
      name: mongo-deployment
    patch: |-
      - op: remove
        path: /spec/template/metadata/labels/org

## Aplica los cambios realizados
kubectl apply -f /root/code/k8s/
deployment.apps/api-deployment created
deployment.apps/mongo-deployment created
service/mongo-cluster-ip-service created
error: error validating "/root/code/k8s/kustomization.yaml": error validating data: [apiVersion not set, kind not set]; if you choose to ignore these errors, turn validation off with --validate=false

## Contenido del fichero api-deployment
nano api-depl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: nginx
          image: nginx
        - name: memcached
          image: memcached

## Contenido del fichero mongo-deployment
nano mongo-depl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: mongo
  template:
    metadata:
      labels:
        component: mongo
        org: KodeKloud
    spec:
      containers:
        - name: mongo
          image: mongo

## Contenido del fichero mongo-deployment
nano mongo-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo-cluster-ip-service
spec:
  type: ClusterIP
  selector:
    component: mongo
  ports:
    - port: 27017
      targetPort: 27017
