## 68.1º Las cuentas disponibles en Kubernetes pueden ser:

## 68.2º Cuenta de usuario: Es la cuenta que usa los administradores, desarrolladores, técnicos de mantenimiento.

## 68.3º Cuenta de servicio: Es la que usa el sistema de kubernetes, las aplicaciones, los complementos, la telemetría y los dispositivos conectados.

## 68.4º Los comandos para consultar los servicios de cuentas son:
kubectl get serviceaccount # --> Consultar el servicio activo
kubectl describe serviceaccount default # --> Describir el servicio marcado
kubectl describe pod ["pod_name"] # --> describir el pod marcado

## 68.5º Las cuentas de servicio se abastecen de tokens mientras trabajan en el servidor, se usa para demostrar la identidad de la cuenta y recibir el acceso
kubectl exec -it ["pod_name"] ls /var/run/secrets/kubernetes.io/serviceaccount # --> consutar el estado del token

## 68.6º Las cuentas de servicio se pueden crear con un comando imperativo, fichero.yaml y comando declarativo
kubectl create serviceaccount ["service_name"] # --> forma imperativa

nano service-definition.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: default
automountServiceAccountToken: false / true # --> parámetro opcional para alternar si queremos auto-montar el token, los token caducan cuando se borra el objeto
kubectl create -f service-definition.yaml # --> forma declarativa

kubectl get serviceaccount # --> consultar las cuentas de servicio

kubectl describe serviceaccount ["service_name"] # --> describir las características del servicio de cuenta

nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
kubectl apply -f pod-definition.yaml

kubectl describe pod my-kubernetes-dashboard # --> describir el pod con el servicio de cuenta asociado

## 68.7 Los token también se pueden crear manualmente con el comando
kubectl create token ["serviceaccount_name"] # --> el contenido del token se muestra desde la salida predeterminada de la consola de comandos, si necesitamos redirigir la salia usaremos:
kubectl create token ["serviceaccount_name"] > ["token_file.txt"] # --> crea un fichero.txt con el contenido del token, este método puede sobreescribir un fichero con el mismo nombre. Viene bien verificar si estamos seguros de realizar este comando
Si necesitamos modificar la duración del token se le añade la opción de duración y las horas
kubectl create token ["serviceaccount_name"] --duration 2h # --> el token tiene 2 horas de duración

## 68.8º El token se puede decodificar para obtener más detalles
jq -R 'split(".") | select(length > 0) | .[0],.[1] | @base64 | fromjson' <<< ["Token_Raw"]

## 68.9º El token se puede incluir en una url con el comando curl
curl https://192.168.56.70:6443/api -insecure --header "Authorization: Bearer ["Token_Raw"]"

## 68.10º Objetivos de las cuentas del servicio:
- Las cuentas de Servicio son usadas por otras aplicaciones o servicios que están interactuando con Kubernetes
- Los tokens son creados por cuentas de servicio para identificarse
- Para usar la Cuenta de Servicio desde una aplicación externa se necesita crear el token manualmente
- Cada Nombre_de_Espacio lleva por defecto una cuenta de servicio
- La cuenta de Servicio predeterminada es enlazada con la creación del pod
- Para enlazar la cuenta de servicio al pod, usamos el campo ["serviceAccountName"] en el fichero.yaml
- Cuando el servicio es enlazado al pod:
-- Se crea automáticamente un token y se monta como un volumen proyectado
-- El token es automático
-- El token expira automáticamente cuando el pod es borrado

##
kubectl get serviceaccounts 
NAME      AGE
default   8m13s
dev       29s

##
kubectl describe deployments.apps | grep Image
    Image:      gcr.io/kodekloud/customimage/my-kubernetes-dashboard

##
kubectl get pods -o yaml | grep serviceAccountName
    serviceAccountName: default

##
kubectl describe pod web-dashboard-6b774f6458-x7ccn | grep -A3 Mounts
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9l2pt (ro)
Conditions:
  Type                        Status

## 
nano dashboard-sa_file.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-sa
  namespace: default
automountServiceAccountToken: true
kubectl create -f dashboard-sa_file.yaml 
serviceaccount/dashboard-sa created

##
kubectl create token dashboard-sa > dashboard_sa_token_file.txt

##
kubectl edit deployments.apps web-dashboard 
spec.template.spec.serviceAccountName: dashboard-sa # --> Localizar e insertar el parámetro y el valor necesario
deployment.apps/web-dashboard edited

##
kubectl edit serviceaccounts dashboard-sa
automountServiceAccountToken: false # --> el servicio de cuenta se creó con el parámetro en ["true"], lo necesitamos con ["false"]
serviceaccount/dashboard-sa edited

##
nano ~/web-dashboard/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-dashboard
  namespace: default
  labels:
    name: web-dashboard
spec:
  replicas: 1
  selector:
    matchLabels:
      name: web-dashboard
  template:
    metadata:
      labels:
        name: web-dashboard
    spec:
      containers:
      - name: web-dashboard
        image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
        ports:
        - containerPort: 8080
        env:
        - name: PYTHONUNBUFFERED
          value: "1"
        volumeMounts:
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount/
          name: token
          readOnly: true
      serviceAccountName: dashboard-sa
      automountServiceAccountToken: false
      volumes:
      - name: token
        projected:
          sources:
          - serviceAccountToken:
              path: token
kubectl apply -f ~/web-dashboard/deployment.yaml
Warning: resource deployments/web-dashboard is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/web-dashboard configured

##
kubectl exec $(kubectl get pod -l name=web-dashboard -o jsonpath='{.items[0].metadata.name}') -- ls /var/run/secrets/kubernetes.io/serviceaccount/
token
