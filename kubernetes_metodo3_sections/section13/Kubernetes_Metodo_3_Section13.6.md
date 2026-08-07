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
- namePrefix/Sufix: añade un prefijo o sufijo a todos los nombres de resursos
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

## 110.7º Muestra de kustomization enlazado con el fichero de servicios
nano kustomization.yaml
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

## 110.9º Muestra del kustomization.yaml
namespace: lab
