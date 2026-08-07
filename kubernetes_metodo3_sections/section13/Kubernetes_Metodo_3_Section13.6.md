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

## 110.4º Muestra de fichero db-service.yaml
nano db-service.yaml
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


