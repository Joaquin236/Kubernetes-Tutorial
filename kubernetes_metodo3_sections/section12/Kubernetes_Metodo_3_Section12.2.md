## Instala el Helm en el servidor
wget --show-progress -vd https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
ls -lh
total 12K
-rw-r--r-- 1 root root 12K Aug  1 16:01 get-helm-3
chmod -v +x get-helm-3
mode of 'get-helm-3' changed from 0644 (rw-r--r--) to 0755 (rwxr-xr-x)
./get-helm-3

## Consulta la versión del Helm instalado
helm version
version.BuildInfo{Version:"v3.21.3", GitCommit:"1ad6e68924fdf6fb0c7dcef8e9e1dfc0f36eaed6", GitTreeState:"clean", GoVersion:"go1.26.5"}

## Crea un archivo para obtener el manual del helm
helm > helm_help.txt
ls -hl
total 24K
-rwxr-xr-x 1 root root  12K Aug  1 16:01 get-helm-3
-rw-r--r-- 1 root root 8.2K Aug  1 16:11 helm_help.txt

## Filtra el contenido y localiza el $HELM_DEBUG
grep "$HELM_DEBUG" helm_help.txt 
The Kubernetes package manager

Common actions for Helm:

- helm search:    search for charts
- helm pull:      download a chart to your local directory to view
- helm install:   upload the chart to Kubernetes
- helm list:      list releases of charts

Environment variables:

| Name                               | Description                                           |
|------------------------------------|-------------------------------------------------------|
| $HELM_DEBUG                        | indicate whether or not Helm is running in Debug mode |

## Filtra el contenido y localiza el --debug
grep "[--debug]" helm_help.txt
--debug                           enable verbose output

## Sustituye el contenido del fichero helm_help.txt
helm get > helm_help.txt 

## Localizar el arg notes
grep notes helm_help.txt 
- The notes provided by the chart of the release
  notes       download the notes for a named release

## Localizar el arg manifest
grep manifest helm_help.txt 
- The generated manifest file
  manifest    download the manifest for a named release

## Localizar el arg hooks
grep hooks helm_help.txt 
- The hooks associated with the release
  hooks       download all hooks for a named release

## Localizar el arg values
grep values helm_help.txt 
- The values used to generate the release
  values      download the values file for a named release

## Localizar el arg output
grep output helm_help.txt 
      --debug                           enable verbose output

## La secuencia de filtrado determina que no existe la opcion o argumento output en Helm