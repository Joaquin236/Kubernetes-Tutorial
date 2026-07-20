## 69.1º Las imagenes de los contenedores también necesitan ajustes de seguridad y repositorio seguro. El valor de la clave image de los pods tiene como fuente un contenedor docker; ["docker.io/library/nginx"]. Google ofrece el suyo propio; ["gcr.io/kubernetes-e2e-test-images/dnsutils"].

## 69.2º Todo el contenido de la imagen es accesible al público, cuando hay una aplicación crítica en el contenedor, necesitamos mantener el sistema fuera del alcance a los ["Crackers"].

## 69.3º A partir de los comandos de docker y crear una estancia de seguridad en docker se puede aplicar un pod con parámetro de seguridad, creando una estancia privada de docker, asociando la estancia privada a un contenedor, crear un secreto con las credenciales del usuario y crear un pod que llame al secreto.
docker login private-registry.io
docker run private-registry.io/apps/internal-app
kubectl create secret docker-registry regcred \
--docker-server=private-registry.io \
--docker-username=registry-user      \
--docker-password=registry-pass       \
--docker-email=registry-email

nano nginx-pod-secure.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: run private-registry.io/apps/internal-app
  imagePullSecrets:
  - name: regcred
kubectl create -f nginx-pod-secure.yaml

## En el menú de ayuda del kubeclt create secret, buscar el dokcer-registry
kubectl create secret --help | grep docker
 A docker-registry type secret is for accessing a container registry.
  docker-registry   Create a secret for use with a Docker registry
  kubectl create secret (docker-registry | generic | tls) [options]

## Localizar la imagen del deploy activo
kubectl describe deployments.apps | grep Ima
    Image:         nginx:alpine

## Obtener el yaml del deploy web, editar el fichero yaml, insertar el nuevo parámetro de imagen
kubectl get deployments.apps web -o yaml > deploy_nginx_file.yaml
nano deploy_nginx_file.yaml
spec.template.spec.containers.-image:myprivateregistry.com:5000/nginx:alpine
kubectl apply -f deploy_nginx_file.yaml
Warning: resource deployments/web is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/web configured

## Consultar los pods después del nuevo despliegue
kubectl get pods
|NAME                  | READY |  STATUS        | RESTARTS  | AGE|
|----------------------|-------|----------------|-----------|----|
|web-587b875755-lbw6p  | 0/1   |  ErrImagePull  | 0         | 59s|
|web-5d4f7c4d94-9sqx6  | 1/1   |  Running       | 0         | 19m|
|web-5d4f7c4d94-trcv9  | 1/1   |  Running       | 0         | 19m|

## Crear el secreto con los parámetros de la conexión para privatizar el contenedor
kubectl create secret docker-registry private-reg-cred --docker-server=myprivateregistry.com:5000       
--docker-username=dock_user --docker-password=dock_password --docker-email=dock_user@myprivateregistry.com
secret/private-reg-cred created

## El fichero deploy_nginx_file.yaml necesita el imagePullSecrets con el nombre del secreto
nano deploy_nginx_file.yaml
spec.template.spec:imagePullSecrets:- name: private-reg-cred
kubectl apply -f deploy_nginx_file.yaml
deployment.apps/web configured

## Consultar los pods del ns default
kubectl get pods -o wide 
|NAME                 |  READY  | STATUS  |  RESTARTS |  AGE  |  IP          |  NODE         |  NOMINATED NODE  | READINESS GATES|
|---------------------|---------|---------|-----------|-------|--------------|---------------|------------------|----------------|
|web-7f9c49cfd8-gr5r7 |  1/1    | Running |  0        |  109s |  10.244.0.10 |  controlplane |  ["none"]        |   ["none"]     |
|web-7f9c49cfd8-jm9qr |  1/1    | Running |  0        |  110s |  10.244.0.9  |  controlplane |  ["none"]        |   ["none"]     |
