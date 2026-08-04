## 100.1º La opción de ayuda del helm nos muestra todas las opciones válidas del comando ["helm --help"]. Para restaurar un despliegue fallido por actualización defectuosa necesitamos usar ["helm rollback"]. Con el comando [helm repo --help] muestra la ayuda para los repositorios. El comando ["helm repo update --help"] muestra la ayuda para actualizar un repositorio que necesite nuevas versiones. Con el comando ["helm search hub/repo wordpress"] realiza la búsqueda de un servicio desplegable

## 100.2º Para realizar una instalación del repositorio necesitamos añadirlo
helm repoadd bitnami https://charts.bitnami.com/bitnami
helm install my-release bitnami/wordpress
## Para listar el despliegue activo usamos un listado
helm list
## Para borrarlo usamos
helm unistall my-release
## Con el comando ["helm repo"] muestra la ayuda de los comandos válidos para la opción repo. Con el comando ["helm repo list"] mostramos los repo activos. Con el comando ["helm repo update"] actualizamos el repositorio. 

## 101.1º También se puede personalizar la plantilla con el comando helm
helm install --set wordpressBlogName="Helm_Tutorials" my-release bitnami/wordpress
             --set wordpressEmail="john@example.com"
## También se puede redirigir los valores a un fichero para evitar escribirlos en el comando o cuando hay muchos parámetros para establecer
nano custom-values.yaml
wordpressBlogName: Helm_Tutorials
wordpressEmail: john@example.com
helm install --values custom-values.yaml
## Para descargar un despliegue y descomprimir usamos
helm pull bitanmi/wordpress
helm pull --untar bitanmi/wordpress
## El contenido se encuentra en el directorio donde se ejecutó el comando, si añadimos ls-lh listamos el contenido descomprimido. Con el comando ["cd wordpress"] visitamos el contenido descomprimido y se puede hacer una instalación local sin descargar nada
helm install ./wordpress