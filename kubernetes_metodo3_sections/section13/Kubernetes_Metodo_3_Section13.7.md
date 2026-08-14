## Listar el directorio /root/ con tree
 tree /root/
/root/
└── code
    ├── README.md
    └── k8s
        ├── db
        │   ├── NoSql
        │   │   ├── db-depl.yaml
        │   │   ├── db-service.yaml
        │   │   └── kustomization.yaml
        │   ├── Sql
        │   │   ├── db-depl.yaml
        │   │   ├── db-service.yaml
        │   │   └── kustomization.yaml
        │   ├── db-config.yaml
        │   └── kustomization.yaml
        ├── kustomization.yaml
        ├── monitoring
        │   ├── grafana-depl.yaml
        │   ├── grafana-service.yaml
        │   └── kustomization.yaml
        └── nginx
            ├── kustomization.yaml
            ├── nginx-depl.yaml
            └── nginx-service.yaml

7 directories, 16 files

## Localizar el valor de la clave sandbox
grep sandbox /root/code/k8s/kustomization.yaml 
  sandbox: dev

## Filtrar el valor de la clave namePrefix
grep namePrefix /root/code/k8s/db/kustomization.yaml 
namePrefix: data-

## Localizar el espacio de nombre logging
grep logging /root/code/k8s/monitoring/kustomization.yaml 
namespace: logging

## Edita el fichero de customización para ofrecer los recursos necesarios desde el monitoring
nano /root/code/k8s/monitoring/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - grafana-depl.yaml
  - grafana-service.yaml

namespace: logging

commonAnnotations:
  owner: bob@gmail.com

## Edita el fichero de customización para ofrecer los recursos necesarios desde el nginx
nano /root/code/k8s/nginx/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - nginx-depl.yaml
  - nginx-service.yaml

commonAnnotations:
  owner: bob@gmail.com

## Compila el directorio y aplica los cambios
kustomize build /root/code/k8s/ | kubectl apply -f -
# Warning: 'commonLabels' is deprecated. Please use 'labels' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
configmap/data-db-credentials unchanged
service/data-mongo-service unchanged
service/data-postgres-service unchanged
service/nginx-service configured
deployment.apps/data-mongo-deployment unchanged
deployment.apps/data-postgres-deployment unchanged
deployment.apps/nginx-deployment configured
Error from server (NotFound): error when creating "STDIN": namespaces "logging" not found
Error from server (NotFound): error when creating "STDIN": namespaces "logging" not found

## Crea el fichero de customización en la raíz del directorio para los nuevos recursos
nano /root/code/k8s/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/
  - monitoring/
  - nginx/

commonLabels:
  sandbox: dev

images:
  - name: postgres
    newName: mysql

## Compila el directorio y aplica los cambios
kustomize build /root/code/k8s/ | kubectl apply -f -
# Warning: 'commonLabels' is deprecated. Please use 'labels' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
configmap/data-db-credentials unchanged
service/data-mongo-service unchanged
service/data-postgres-service unchanged
service/nginx-service unchanged
deployment.apps/data-mongo-deployment unchanged
deployment.apps/data-postgres-deployment unchanged
deployment.apps/nginx-deployment unchanged
Error from server (NotFound): error when creating "STDIN": namespaces "logging" not found
Error from server (NotFound): error when creating "STDIN": namespaces "logging" not found

## Añade las imagenes del servidor web y motor de la base de datos
nano /root/code/k8s/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/
  - monitoring/
  - nginx/

commonLabels:
  sandbox: dev

images:
  - name: postgres
    newName: mysql

images:
  - name: nginx
    newTag: 1.23

##
nano /root/code/k8s/nginx/kustomization.yaml 
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - nginx-depl.yaml
  - nginx-service.yaml

commonAnnotations:
  owner: bob@gmail.com

images:
  - name: nginx
    newName: nginx
    newTag: "1.23"

## Compila y aplica los cambios
kustomize build /root/code/k8s/ | kubectl apply -f -
# Warning: 'commonLabels' is deprecated. Please use 'labels' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
configmap/data-db-credentials unchanged
service/data-mongo-service unchanged
service/data-postgres-service unchanged
service/nginx-service unchanged
deployment.apps/data-mongo-deployment unchanged
deployment.apps/data-postgres-deployment configured
deployment.apps/nginx-deployment unchanged
Error from server (NotFound): error when creating "STDIN": namespaces "logging" not found
Error from server (NotFound): error when creating "STDIN": namespaces "logging" not found