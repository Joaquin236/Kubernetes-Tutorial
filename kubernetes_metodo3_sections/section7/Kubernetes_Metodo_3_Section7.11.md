## 71.1º Para aplicar los contextos de seguridad en kubernetes, se realizará un fichero yaml con un pod y los parámetros de seguridad adaptados a kubernetes.
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
      - name: ubuntu
        image: ubuntu
        command: ["sleep","3600"]
        securityConext:
          runAsUser: stand_user
          capabilities:
             add: ["MAC_ADMIN"]
kubectl create -f pod-definition.yaml

## Ejecutar un comando dentro del contenedor de un pod
kubectl exec pods/ubuntu-sleeper -- whoami
root

## Exportar el pod ubuntu-sleepr en un yaml para editar, añadimos los parámetros para añadir el usuario y reemplazar el pod
kubectl get pods ubuntu-sleeper -o yaml > ubuntu-sleeper_pod.yaml
nano ubuntu-sleeper.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:      # Update securityContext
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
kubectl delete pod ubuntu-sleeper ; kubectl create -f ubuntu-sleeper_pod.yaml
pod "ubuntu-sleeper" deleted from default namespace
pod/ubuntu-sleeper created

## Analiza el fichero multi-pod.yaml y localiza los usuarios que se ejecutará
nano multi-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]

## ¿Que usuario ejecutará el pod con el nombre de ["web"]?
grep runAsUser multi-pod.yaml 
    runAsUser: 1001
      runAsUser: 1002 # --> this is the answer

## ¿Que usuario ejecutará el pod con el nombre de ["sidecar"]?
grep runAsUser multi-pod.yaml 
    runAsUser: 1001 # --> this is the answer
      runAsUser: 1002

## Modifica el pod para que ofrezca la capacidad de usar el ["SYS_TIME"], el pod debe ser remplazado para el nuevo despliegue
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:        # Updated securityContext on container level
      capabilities:
        add: ["SYS_TIME"]
kubectl delete pod ubuntu-sleeper ; kubectl create -f ubuntu-sleeper_pod.yaml
pod "ubuntu-sleeper" deleted from default namespace
pod/ubuntu-sleeper created

## Agrega en la misma linea el acceso a NET_ADMIN
nano ubuntu-sleeper_pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:        # Updated securityContext on container level
      capabilities:
        add: ["SYS_TIME","NET_ADMIN"]
kubectl delete pod ubuntu-sleeper ; kubectl create -f ubuntu-sleeper_pod.yaml
pod "ubuntu-sleeper" deleted from default namespace
pod/ubuntu-sleeper created