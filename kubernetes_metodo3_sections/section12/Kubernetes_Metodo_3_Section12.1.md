## 95.1 º Normalmente se suele usar distintos ficheros yaml para realizar un despliege, si se hace más de tres despliegues para una infraestructura hay que usar el comando [kubectl_create/apply_-f_deploy_file.yaml] varias veces seguidas. A parte de guardar muchos ficheros similares.

## 95.2º Muestra de un fichero de secretros
nano wordpress_secrect.yaml
apiVersion: v1
kind: Secret
metadata:
  name: wordpress-admin-password
data:
  key: [base64_code]
kubectl apply -f wordpress_secrect.yaml

## 95.3º Muestra de un fichero de servicios
nano wordpress_service.yaml
apiVersion: v1
kind: Service
metadata: 
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
kubectl apply -f wordpress_service.yaml

## 95.4º Muestra de un fichero de despliegue
nano wordpress_deploy.yaml
apiVersion: v1
kind: Deployment
metadata: 
  name: wordpress
  labels:
    app: wordpress
spec:
  matchLabels:
    app: wordpress
    tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
kubectl apply -f wordpress_deploy.yaml

## 95.6º Muestra de un fichero de reclamar volumen
nano wordpress_persistent_vol_claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  name: wd-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
kubectl apply -f wordpress_persistent_vol_claim.yaml

## 95.7º Muestra de fichero de volumen persistente
nano wordpress_persistent_vol.yaml
apiVersion: v1
kind: PersistentVolume
metadata: 
  name: pv0003
  labels:
    app: wordpress
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
kubectl apply -f wordpress_persistent_vol.yaml

## 95.8º Este proceso si se incrementa el número de ficheros empieza a ser inconsistente porque hay que aplicar cada fichero uno por uno. Una solucón es fusionar todo en un solo fichero y aplicar una vez. Pero tener un solo fichero equivale a buscar línea por línea el error de sintaxis o un bug en las directivas.

## 96.1º La solución más práctica es la herramienta Helm, Kubernetes no verifica la aplicación del adminsitrador. Solo interpreta la declaración de objetos y los crea para alojarlos en el cluster. No evalúa que aplicaciones se están ejecuando dentro del servicio, despliegue, volumen persistente, Pod/Contenedor o en el nodo.

## 96.2º La herramienta Helm si evalúa la aplicación que se encuentra en el cluster, también se le conoce como gestor de paquetes para Kubernetes. 

## 96.3º Con un solo comando se puede instalar toda la aplicación. Realiza un despliegue automático de diveros objetos. Algunos de sus comandos son
helm install wordpress  #--> Instalar wordpress
helm upgrade wordpress  #--> Actualizar wordpress
helm rollback wordpress #--> Restaurar wordpress
helm unistall wordpress #--> Borrar wordpress

## 96.4º Para instalar el Helm se necesita un cluster de kuerbenetes operativo, admite multiplataforma. Estos son los comandos Linux para instalarlo
## Instalación dede snap
sudo snap install helm --classic
## Instalación desde un script descargable
wget --show-progress -vd https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
ls -lh
total 12K
-rw-r--r-- 1 root root 12K Aug  1 16:01 get-helm-3
chmod -v +x get-helm-3
mode of 'get-helm-3' changed from 0644 (rw-r--r--) to 0755 (rwxr-xr-x)
./get-helm-3