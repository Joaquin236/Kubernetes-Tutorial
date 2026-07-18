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

## Localizar los roles activos
kubectl get roles  --all-namespaces 
NAMESPACE     NAME                                             CREATED AT
kube-public   system:controller:bootstrap-signer               2026-07-18T14:23:34Z
kube-system   extension-apiserver-authentication-reader        2026-07-18T14:23:34Z

kubectl get rolebindings.rbac.authorization.k8s.io  --all-namespaces 
NAMESPACE     NAME                                                 ROLE                                                  AGE
kube-public   system:controller:bootstrap-signer                   Role/system:controller:bootstrap-signer               21m
kube-system   k3s-cloud-controller-manager-authentication-reader   Role/extension-apiserver-authentication-reader        21m

## Localizar los enlaces activos
kubectl describe clusterrolebindings.rbac.authorization.k8s.io cluster-admin 
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name            Namespace
  ----   ----            ---------
  Group  system:masters  

## Describir el rol de cluster para el cluster-admin
kubectl describe clusterrole cluster-admin 
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]

## Crear un fichero para el rol con el usuario michelle
nano michelle_role_file_1.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: node-admin
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get","watch","list","create","delete"]

## Crear un fichero para enlazar el rol con el usuario michelle
nano michelle_rolebings_file_1.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-binding
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-admin
  apiGroup: rbac.authorization.k8s.io

## Crear el rol y el enlace para el ususario michelle
kubectl create -f michelle_role_file_1.yaml ; kubectl create -f michelle_rolebings_file_1.yaml 
clusterrole.rbac.authorization.k8s.io/node-admin created
clusterrolebinding.rbac.authorization.k8s.io/michelle-binding created

## Localizar el masters en la descripción del clusterrolebindings en todos los ns
kubectl describe clusterrolebindings.rbac.authorization.k8s.io --all-namespaces | grep masters
  Group  system:masters  

## Crear un segundo fichero para el rol del usuario michelle
nano michelle_role_file_2.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: storage-admin
rules:
- apiGroups: [""]
  resources: ["persistentvolumes"]
  verbs: ["get","watch","list","create","delete"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get","watch","list","create","delete"]

## Crea un segundo fichero y el enlace para el ususario michelle
nano michelle_rolebindings_file_2.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: michelle-storage-admin
subjects:
- kind: User
  name: michelle
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: storage-admin
  apiGroup: rbac.authorization.k8s.io

## Crear el segundo rol y enlace para el usuario michelle
kubectl create -f michelle_role_file_2.yaml ; kubectl create -f michelle_rolebindings_file_2.yaml
clusterrole.rbac.authorization.k8s.io/storage-admin created
clusterrolebinding.rbac.authorization.k8s.io/michelle-storage-admin created
