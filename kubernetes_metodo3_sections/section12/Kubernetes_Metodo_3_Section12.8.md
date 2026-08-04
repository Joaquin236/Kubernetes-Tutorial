## Agrega el repositorio de bitnami al sistema 
helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

## Listar el despliegue activo
helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
dazzling-web    default         3               2026-08-04 16:51:48.279490618 +0000 UTC deployed        nginx-12.0.4    1.22.0     

## Actualizar el despliegue a la versión 18.3.6 y verifica que versión de nginx se ha recibido
helm upgrade dazzling-web bitnami/nginx --version 18.3.6
Pulled: us-central1-docker.pkg.dev/kk-lab-prod/helm-charts/bitnami/nginx:18.3.6
Digest: sha256:19a3e4578765369a8c361efd98fe167cc4e4d7f8b4ee42da899ae86e5f2be263
Release "dazzling-web" has been upgraded. Happy Helming!
NAME: dazzling-web
LAST DEPLOYED: Tue Aug  4 16:56:51 2026
NAMESPACE: default
STATUS: deployed
REVISION: 4
TEST SUITE: None
NOTES:
CHART NAME: nginx
CHART VERSION: 18.3.6
APP VERSION: 1.27.4

## Hay un aviso del desarrollador de que la actualización no está optimizada para el sistema, se solicita un rollback
helm rollback dazzling-web
Rollback was a success! Happy Helming!
