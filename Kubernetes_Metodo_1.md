# IMPLEMENTACIÓN DE EJEMPLO. Método 1 --> 
Video_Tutorial de ejemplo --> https://www.youtube.com/watch?v=DzuU1KLQ240&t=902s

## 1º Descarga el k3s.io
curl -sfL https://get.k3s.io | bash - 

## 2.1º Descarga el kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

## 2.2º Instala el kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

## 2.3º Verifica el estado de kubectl
kubectl version --client --output=yaml 

## 3º Crea una plantilla de configuración, el contenido debe ser redirigido
sudo k3s kubectl config view --raw > ~/.config_k3s.yaml

## 4º Crea una variable con la ruta de fichero
export KUBECONFIG=~/.config_k3s.yaml 

## 5º Muestra todos los cluster activos
kubectl get node

## 6º Muestra los pods activos
kubectl get pods

## 7º Descarga el script que instala el helm, un gestor para kubernetes
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod +x -v get_helm.sh
./get_helm.sh

## 8º Muestra los comandos válidos para el helm
helm

# 9.1º Buscar packs para instalar (el enlace se muestra cortado con puntos suspensivos)
helm search hub podinfo

# 9.2º Instalar los charts
helm install my-podinfo oci://ghcr.io/stefanprodan/charts/podinfo

# 10.1º Muestra los valores del chart antes de instalar
helm show chart oci://ghcr.io/stefanprodan/charts/podinfo

# 10.2º Verificar la instalación del charts
kubectl port-forward svc/my-podinfo 9898:9898 ; curl http://localhost:9898

# 11º Listar las entradas chart activas
helm list

# 12º Mostrar información del pod
helm status my-podinfo

# 13º Entrar en el pod activo
kubectl -n default port-forward deploy/my-podinfo 8080:9898

## Conclusión: Es simple de usar pero no es extenso
