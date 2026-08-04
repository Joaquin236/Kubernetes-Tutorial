## Localizar el hub de wordpress
helm search hub wordpress
URL                                                     CHART VERSION   APP VERSION             DESCRIPTION                                       
https://artifacthub.io/packages/helm/wordpress-...      1.0.3           7.0.2                   WordPress is the world's most popular blogging ...
https://artifacthub.io/packages/helm/slybase-wo...      5.4.0           7.0.1                   Using the official WordPress image. This chart ...
https://artifacthub.io/packages/helm/quench-wor...      0.0.9           7.0.2                   Hardened WordPress CMS (PHP-FPM + nginx) on a 0...

## Localizar la versión 2.0.2 del hashicorp
helm search hub hashicorp | grep 2.0.2
https://artifacthub.io/packages/helm/hashicorp/...      2.0.2           2.0.2           Official HashiCorp Consul Chart                   
https://artifacthub.io/packages/helm/vault-helm...      0.4.0           2.0.2           A tool for secrets management, encryption as a ...
https://artifacthub.io/packages/helm/wenerme/co...      2.0.2           2.0.2           Official HashiCorp Consul Chart                   
https://artifacthub.io/packages/helm/wener/consul       2.0.2           2.0.2           Official HashiCorp Consul Chart 

## Añadir el repositorio de bitnami al sistema
helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

## Localizar el repositorio de Wordpress
helm search repo wordpress
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                                       
bitnami/wordpress       33.0.1          7.0.2           WordPress is the world's most popular blogging ...
bitnami/wordpress-intel 2.1.31          6.1.1           DEPRECATED WordPress for Intel is the most popu...

## Listar los repositorios activos en el sistema
helm repo list
NAME            URL                                                 
bitnami         https://charts.bitnami.com/bitnami                  
puppet          https://puppetlabs.github.io/puppetserver-helm-chart
hashicorp       https://helm.releases.hashicorp.com                 

## Instalar en el sistema el amaze-surf con el apache
helm install amaze-surf bitnami/apache
Pulled: us-central1-docker.pkg.dev/kk-lab-prod/helm-charts/bitnami/apache:11.3.2
Digest: sha256:1bd45c97bb7a0000534e3abc5797143661e34ea7165aa33068853c567e6df9f2
NAME: amaze-surf
LAST DEPLOYED: Tue Aug  4 15:19:47 2026
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: apache
CHART VERSION: 11.3.2
APP VERSION: ["2.4.63"]

Did you know there are enterprise versions of the Bitnami catalog? For enhanced secure software supply chain features, unlimited pulls from Docker, LTS support, or application customization, see Bitnami Premium or Tanzu Application Catalog. See https://www.arrow.com/globalecs/na/vendors/bitnami for more information.

** Please be patient while the chart is being deployed **

1. Get the Apache URL by running:

** Please ensure an external IP is associated to the amaze-surf-apache service before proceeding **
** Watch the status using: kubectl get svc --namespace default -w amaze-surf-apache **

  export SERVICE_IP=$(kubectl get svc --namespace default amaze-surf-apache --template "{{ range (index .status.loadBalancer.ingress 0) }}{{ . }}{{ end }}")
  echo URL            : http://$SERVICE_IP/

WARNING: You did not provide a custom web application. Apache will be deployed with a default page. Check the README section "Deploying your custom web application" in https://github.com/bitnami/charts/blob/main/bitnami/apache/README.md#deploying-a-custom-web-application.

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

## Localiza la versión del amaze-surf
helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
amaze-surf      default         1               2026-08-04 15:19:47.749986497 +0000 UTC deployed        apache-11.3.2   ["2.4.63"]     
crazy-web       default         1               2026-08-04 15:21:11.050006324 +0000 UTC deployed        nginx-19.0.0    1.27.4     
happy-browse    default         1               2026-08-04 15:21:07.880567826 +0000 UTC deployed        nginx-19.0.0    1.27.4

## Localiza los pods con nginx
kubectl get pods -o wide | grep nginx
crazy-web-nginx-d747d6768-vpxr7      0/1     Init:ErrImagePull       0          101s   172.17.0.7   controlplane   [none]           [none]
happy-browse-nginx-768c7765c-867p4   0/1     Init:ImagePullBackOff   0          104s   172.17.0.6   controlplane   [none]           [none]

## Localiza los pods con happy
kubectl get pods -o wide | grep happy
happy-browse-nginx-768c7765c-867p4   0/1     Init:ImagePullBackOff   0          2m37s   172.17.0.6   controlplane   [none]           [none]

## Borra el despliegue del happy-browse
helm uninstall happy-browse
release "happy-browse" uninstalled

## Listar el repositorio de hashicorp
helm repo list | grep hashicorp
hashicorp       https://helm.releases.hashicorp.com                 

## Borrar el Hashicorp
helm repo remove hashicorp
"hashicorp" has been removed from your repositories
