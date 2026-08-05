## 105.1º Para instalar Kustomize se necesita un cluster de Kubernetes operativo, el acceso al comando kubectl, la comunidad de Kustomize ha diseñado un script para evaluar los requisitos necesarios, neceistamos usar curl para descargarlo
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash

## Si se ha instalado bien el comando puede realizar acciones como mostrar la versión de aplicación
kustomize version

## 106.1º El formato de estructura del Kustomize sigue este patrón
K8s/
├── nginx-depl.yaml
├── nginx-service.yaml
└── kustomization.yaml

## Interior del fichero Kustomization.yaml
# kubernetes resources to be managed by kustomize
resources:
    nginx-deployment.yaml
    nginx-service.yaml
# Customizations that need to be made
commonLabels:
  company: KodeKloud

## 106.2º El comando ["kustomize build k8s/"] crea un fichero yaml con lo esté definido en el directorio marcado

## 107.3º Los aspectos a tener en cuenta sobre el Kustomize son:
- Kustomize evalúa el fichero kustomization y su contenido:
- Realiza un listado de los manifiestos de Kubernetes y administrar con Kustomize
- Toda la personalización/customización que se vaya a aplicar
- El comando ["kustomize buid"] combina todos los manifiestos y aplica la transformación definida
- El comando no aplica/despliega los recursos al cluster de kubernetes
- La salida necesita redirigirla al comando ["kubectl apply -f 'manifest_file.yaml'"]
