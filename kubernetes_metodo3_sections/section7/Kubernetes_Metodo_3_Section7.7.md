## 67.1º Los roles y el enlace de los roles se establecen dentro del namespace, cuando no se establece siempre elige el default, los recursos de ámbito son los que no se establece el namespace al crearlos: ["nodes","volumen_persistente","roles","enlace_de_rol","respuestas_certificado_firmado","namespace"]. Los objetos ns no tienen espacios de nombres, es importante saber que no es necesario una lista de recursos. Para ver la lista completa de ns y recursos que no son espacios de nombres. Usamos los comandos:
kubectl api-resources --namespace=true
kubectl api-resources --namespace=false

## 67.2º Para crear un rol de administrar el cluster para ofecer un administrador los permisos para consultar borrar y crear los nodos en un cluster. Se puede crear un rol de almacenamiento para autorizar el acceso del usuario administrador para crear volumen persistente.

## 67.3º La definición del rol de cluster se realiza desde un fichero yaml donde se declara los valores correspondientes:

nano cluster-admin-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list","get","create","delete"]
kubectl create -f cluster-admin-role.yaml

El fichero depende de un enlace de roles;
nano cluster-admin-role-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
- kind: User
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
kubectl create -f cluster-admin-role-binding.yaml

##