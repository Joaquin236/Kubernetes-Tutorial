## 103.1º La herramienta Kustomize permite personalizar algunos aspectos del despliegue de Kubernetes. En un desliegue, hay tres directiorios con un fichero cada uno. ["dev:nginx-depl.yaml","stg:nginx-delp.yaml","prod:nginx-delp.yaml"]. Las diferencias entre los ficheros son: el número de replicas que despliegan los ficheros. Para aplicar este formato de despliegue establecemos el nombre del directorio kubectl apply -f ["dev/","stg/","prod/"]. Hay que repetir tres veces por cada directorio para completar el despliegue de esta estructura. 

## 103.2º La estructura puede mejorar y avanzar, durante el desarrollo se puede crear ficheros nuevos, se va a agregar un fichero de servcio. El service.yaml, a largo plazo esta forma de desarrollar no es totalmente escalable.

## 103.3º Las superposiciones nos ofrecerán personalizar el sistema en función del entorno, tenemos un fichero de despliegue como base, después establecemos ficheros de overlays/["dev","stg","prod"] con el contenido de las replicas.

## 103.4º Este listado describe las capacidades del Kustomize 
- Kustomize crea despliegues con kubectl sin necesidad de instalar paquetes a parte.
- Si quieres instalar el kustomize-cli para obtener la última versión. Kubectl no trabaja con la última versión por ahora.
- No se requiere aprender con complejidad los sistemas de plantillas.
- Cada elemento de Kustomize usa plantillas YAML y puede validar y procesarlas

## 103.5º Muestra del fichero nginx-depl.yaml
nano depl_base_file.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1
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

## 103.6º Si establecemos ficheros de superposición de algunos valores, puede crear y sobreescribir los despliegues y cambiar los valores necesarios
nano overlays_dev.yaml
spec:
  replicas: 1
-------------------
nano overlays_stg.yaml
spec:
  replicas: 2
--------------------
nano overlays_prod.yaml
spec:
  replicas: 5
-------------------
base + overlay = final_manifest

## 103.7º Estructura de directorios del entorno
Env_dir/
├── dev/
├── stg/
└── prod/

## 103.8º Estructura con directorio y fichero juntos
Env_dir/
├── dev/
│   ├─── depl_file.yaml
│   ├─── service_file.yaml
├── stg/
│   ├─── depl_file.yaml
│   ├─── service_file.yaml
└── prod/
    ├─── depl_file.yaml
    ├─── service_file.yaml

## 103.9º Estructura de un despliegue de K8S
k8s/
├─── base/ # --> Comparte o cruzar los valores predefinidos al entorno
│    ├─── kustomization.yaml
│    ├─── nginx-depl.yaml
│    ├─── service.yaml
│    └─── redis-depl.yaml
└─── overlays/ # --> Entorno de configuración que añade o modifica la base de configuración
     ├─── dev/
     |    ├─── kustomization.yaml
     |    └─── config-map.yaml
     |
     ├─── stg/
     |    ├─── kustomization.yaml
     |    └─── config-map.yaml
     └─── prod/
          ├─── kustomization.yaml
          └─── config-map.yaml
