## 114.1º Otras opciones del Kustomize es compartir las configuraciones por defecto cruzadas en los entornos que estén configurandose. El subdirectorio Overplays especifica las configuraciones a añadir o modificar de la estructura base
k8s/
├── base/
|   ├── kustomization.yaml
|   ├── nginx-depl.yaml
|   ├── service.yaml
|   └── redis-depl.yaml
└── overplays/
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

## 114.4º 


