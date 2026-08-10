## 114.1º Otras opciones del Kustomize es compartir las configuraciones por defecto cruzadas en los entornos que estén configurandose. El subdirectorio Overlays especifica las configuraciones a añadir o modificar de la estructura base
k8s/
├── base/
|   ├── kustomization.yaml
|   ├── nginx-depl.yaml
|   ├── service.yaml
|   └── redis-depl.yaml
└── overlays/
    ├── dev/
    |   ├── kustomization.yaml
    |   └── config-map.yaml
    ├── stg/
    |   ├── kustomization.yaml
    |   └── config-map.yaml
    └── prod/
        ├── kustomization.yaml
        └── config-map.yaml

## 114.2º El fichero /root/code/k8s/base/nging-depl.yaml se establece las replicas que debe recibir el despliegue final
nano /root/code/k8s/base/nging-depl.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 1

## 114.3º El fichero de kustomization.yaml lleva las rutas de los ficheros a procesar
nano /root/code/k8s/base/kustomization.yaml
resources:
  - nginx-depl.yaml
  - service.yaml
  - redis-depl.yaml

## 114.4º Este proceso es un proceso estándar, hay más opciones que se pueden aplicar para aumentar las oportunidades de personalizar el despliegue. El fichero /root/code/k8s/overlays/dev/kustomization.yaml realiza el patch al directorio base y reemplaza el valor de las replicas. La ruta se puede establecer en formato relativo o absoluto. 
nano /root/code/k8s/overlays/dev/kustomization.yaml
# Subir hasta la posición del directorio base
bases:
  - ../../base
# Parchear el valor de las replicas del fichero del directorio base
patch: |-
      - op: replace
        path: /spec/replicas
        value: 2

## 114.5º Después de aplicarlo debe haber dos replicas en el sistema

## 114.6º Si aplicamos este otro fichero, se realizará tres replicas reemplazando 
nano /root/code/k8s/overlays/prod/kustomization.yaml
# Subir hasta la posición del directorio base
bases:
  - ../../base  
# Parchear el valor de las replicas del fichero del directorio base
patch: |-
      - op: replace
        path: /spec/replicas
        value: 3

## 114.7º En esta estructura se puede realizar las superposiciones desde el directorio prod con el fichero grafana-depl.yaml que solo se ubica en /root/code/k8s/prod/*
nano /root/code/k8s/overlays/prod/kustomization.yaml
# Subir hasta la posición del directorio base
bases:
  - ../../base  
# Fichero de recursos para procesar                                
resources:
  - grafana-depl.yaml
        
# Parchear el valor de las replicas del fichero base
patch: |-
      - op: replace
        path: /spec/replicas
        value: 2

## 114.8º Se puede añadir todos los recursos necesarios al directorio de la superposición de ficheros. Kustomization ofrece flexibilidad sobre la estructura del despliegue para Kubernetes, el directorio de inicio del proyecto puede recibir varios subdirectorios para clasificar las funciones del despliegue, configuración y parches de corrección de despliegue. 

## 114.9º Nueva estructura del directorio k8s
k8s/
├── base/
|   ├── db/
|   |   ├── db-depl.yaml
|   |   ├── db-service.yaml
|   |   └── kustomization.yaml
|   └── api/
|       ├── api-depl.yaml
|       ├── api-service.yaml
|       └── kustomization.yaml
└── overlays/
    ├── dev/
    |   ├── kustomization.yaml
    |   ├── db/
    |   |   ├── db-patch.yaml
    |   |   └── kustomination.yaml
    |   └── api/
    |       ├── api-patch.yaml
    |       └── kustomination.yaml
    └── prod/
        ├── kustomization.yaml
        ├── db/
        |   ├── db-patch.yaml
        |   └── kustomination.yaml
        └── api/
            ├── api-patch.yaml
            └── kustomination.yaml
