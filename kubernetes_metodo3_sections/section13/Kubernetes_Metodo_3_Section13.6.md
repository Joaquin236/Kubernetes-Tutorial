## 110.1º Kustomize incluye opciones de transformación incorporadas y crear los transformadores del usuario, los que vamos a usar son los transformadores comunes.

## 110.2º En los despliegues y servicios suelen haber claves con los mismos valores y necsitamos establecer una configuración común. En cada fichero del despliegue tiene que haber una etiqueta compartida con la clave y el valor. 

## 110.3º Muestra de fichero db-depl.yaml
nano db-depl.yaml
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: api-deployment-dev
spec:
  replicas: 1
  selector: 
    matchLabels:
      component: api
      org: KodeKloud
  template:
    metadata:
      labels:
        component: api
        org: KodeKloud
    spec:
      containers:
        - name: nginx
          image: nginx

## 110.4º Muestra de fichero db-service-1.yaml
nano db-service-1.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    org: KodeKloud
  name: api-service-dev
spec:
  selector:
    compoment: api
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
      type: LoadBalancer

## 110.5º Listado de atributos del Kustomization_transformers
- commonLabel: añade un label a todos los recursos de Kubernetes
- namePrefix/Suffix: añade un prefijo o sufijo a todos los nombres de resursos
- Namespace: añade un namespace común a todos los recursos
- commonAnnotations: añade una anotación a todos los recursos

## 110.6º Muestra de fichero db-service-2.yaml
nano db-service-2.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    org: KodeKloud
  name: api-service
spec:
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
  selector:
    compoment: api
    org: KodeKloud
  type: LoadBalancer

## 110.7º Muestra de kustomization enlazado el commonLabel con el fichero de servicios-2
nano kustomization-1.yaml
commonLabel:
  org: KodeKloud

## 110.8º Muestra de fichero db-service-3.yaml
nano db-service-3.yaml
apiVersion: v1
kind: Service
metadata:
  annotations: 
    branch: master
  labels:
    org: KodeKloud
  name: api-service
  namespace: lab
spec:
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
    selector:
      compoment: api
      org: KodeKloud
    type: LoadBalancer

## 110.9º Muestra del kustomization.yaml con el namespace enlazando con el fichero de servicios-3
nano kustomization-2.yaml
namespace: lab

## 110.10º Muestra de fichero db-service-4.yaml
nano db-service-4.yaml
apiVersion: v1
kind: Service
metadata:
  name: KodeKloud-service-dev
spec:
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
    selector:
      compoment: api
    type: LoadBalancer

## 110.11º Enlazamos el kustomization con el prefijo y sufijo
nano kustomization-3.yaml
namePrefix: KodeKloud-
nameSuffix: -dev

## 110.12º Muestra de fichero db-service-5.yaml
nano db-service-5.yaml
apiVersion: v1
kind: Service
metadata:
  annotations: 
    branch: master
  labels:
    org: KodeKloud
  name: api-service
  namespace: lab
spec:
  ports:
    - protocol: "TCP"
      port: 80
      targetPort: 3000
    selector:
      compoment: api
      org: KodeKloud
    type: LoadBalancer

## 110.13º El db-service-5 se enlaza con el branch del annotations
nano kustomization-4.yaml
annotations: 
  branch: master

## 111.1º El mismo proceso de los transformers para los ficheros de despliegue se puede aplicar a las img de instalación. En el fichero de kustomization se añade parámetros para reemplazar el nombre que identifica la imagen, se declara el nombre de imagen antiguo y el nombre de imagen nuevo

## 111.2º Muestra de fichero db-depl-2.yaml
nano db-depl-2.yaml
apiVersion: apps/v1
kind: Deployment 
metadata:
  name: api-deployment-dev
spec:
  replicas: 1
  selector: 
    matchLabels:
      component: api
      org: KodeKloud
  template:
    metadata:
      labels:
        component: api
        org: KodeKloud
    spec:
      containers:
        - name: nginx
          image: nginx

## 111.3º El fichero kustomization lleva los nombres de imagen
kustomization-5.yaml
images:
  - name: nginx
    newName: haproxy

## 111.4º Después de este cambio, el deploy que se iba a hacer con Nginx, se transforma en un despliegue con Haproxy. Todo el contenido también puede cambiar.

## 111.5º Si queremos preservar la aplicación y la imagen pero cambiando la versión, tenemos que evitar el cambio de imagen y seleccionar el cambio de versión
kustomization-6.yaml
images:
  - name: nginx
    newTag: 2.4


