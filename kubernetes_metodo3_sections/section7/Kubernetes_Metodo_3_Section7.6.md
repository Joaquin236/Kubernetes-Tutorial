## 66.1º Para crear un control de acceso por roles se establece un fichero yaml con las opciones y el grupo de usuarios que será filtrado. En el fichero el grupo de desarrolladores pueden hacer las acciones de consultar, crear, actualizar y borrar los pods. Mientras solo puede crear el ConfigMap.

nano developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","get","create","update","delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]

Para desplegar el fichero usaremos el comando:
kubectl create -f developer-role.yaml

Para enlazar el usuario necesitamos otro fichero con los parámetros de enclace:

nano devuser-developer-binding.yaml
apiVersion: rbac.authentication.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

El enlace se realiza aplicando con el comando:
kubectl create -f devuser-developer-binding.yaml

## 66.2º Al establecer los parámetros el usuario recibe el rol y puede realizar las acciones permitidas. Según sea necesario, con el tiempo se puede limitar los permisos que se ha concedido.

Con el comando de consultas de RBAC se puede verificar el estado:
kubectl get roles --> Visualiza los roles activos
kubectl get rolebindings --> Visualiza los enlaces activos
kubectl describe role developer --> Visualiza los parámetros y la descipción del rol establecido
kubectl describe rolebinding devuser-developer-binding --> Visualiza los parámetros y la descipción del enlace del usuario hacia el rol establecido

## 66.3º Para verificar el estado de un permiso se usa un comando que pregunta si la acción es válida:
## 1º kubectl auth can-i create deployments --> devuelve yes si está permitido
## 2º kubectl auth can-i delete nodes --> devuelve no cuando no lo tiene permitido
## 3º kubectl auth can-i create deployments --as dev-user --> devuelvo no cuando no está permitido, el parámetro --as user es para consultar otros usuarios del sistema
## 4º kubectl auth can-i create pods --as dev-user --> devuelve yes si está permitido
## 5º kubectl auth can-i create pods --as dev-user --> devuelve null cuando no está establecido y no puede verificar si lo tiene concedido
## 6º kubectl auth can-i create deployments --as dev-user --as namespace test --> devuelvo no cuando no está permitido sobre el namespace establecido y sobre el usuario determinado

## 66.4º Los resource names también pueden interactuar con los ficheros de los roles, se establecen a pie del fichero con el parámetro resourceNames, este rol limita las acciones de consultar, crear y borrar a los namespaces establecidos:

nano developer-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","create","delete"]
  resourceNames: ["blue","orange"]

