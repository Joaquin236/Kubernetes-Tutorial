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
## 5º kubectl auth can-i create deployments --as dev-user -n test --> devuelvo no cuando no está permitido sobre el namespace establecido y sobre el usuario determinado

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

## Descibir los pod de kube-apiserver en el ns kube-system y filtrar por authorization
kubectl describe pod kube-apiserver-controlplane -n kube-system | grep "autho*"
      --authorization-mode=Node,RBAC
      --enable-bootstrap-token-auth=true

## Localizar los roles en el ns default
kubectl get roles.rbac.authorization.k8s.io
No resources found in default namespace.

## Localizar los roles de todos los ns
kubectl get roles.rbac.authorization.k8s.io -A
NAMESPACE     NAME                                             CREATED AT
blue          developer                                        2026-07-17T16:29:54Z
kube-public   kubeadm:bootstrap-signer-clusterinfo             2026-07-17T16:21:27Z
kube-public   system:controller:bootstrap-signer               2026-07-17T16:21:26Z
kube-system   extension-apiserver-authentication-reader        2026-07-17T16:21:26Z
kube-system   kube-proxy                                       2026-07-17T16:21:28Z
kube-system   kubeadm:kubelet-config                           2026-07-17T16:21:27Z
kube-system   kubeadm:nodes-kubeadm-config                     2026-07-17T16:21:27Z
kube-system   system::leader-locking-kube-controller-manager   2026-07-17T16:21:26Z
kube-system   system::leader-locking-kube-scheduler            2026-07-17T16:21:26Z
kube-system   system:controller:bootstrap-signer               2026-07-17T16:21:26Z
kube-system   system:controller:cloud-provider                 2026-07-17T16:21:26Z
kube-system   system:controller:token-cleaner                  2026-07-17T16:21:26Z

## Describir el rol de kube-proxy en kube-system para buscar el resource
kubectl describe role kube-proxy -n kube-system 
Name:         kube-proxy
Labels:       ["none"]
Annotations:  ["none"]
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]

Answer --> ConfigMaps

## Describir el rol de kube-proxy en kube-system para buscar el verbs
kubectl describe role kube-proxy -n kube-system 
Name:         kube-proxy
Labels:       ["none"]
Annotations:  ["none"]
PolicyRule:
  Resources   Non-Resource URLs  Resource Names  Verbs
  ---------   -----------------  --------------  -----
  configmaps  []                 [kube-proxy]    [get]

Answer --> GET

## Describir el acceso al rol de kube-proxy en kube-system
kubectl describe rolebindings.rbac.authorization.k8s.io kube-proxy -n kube-system 
Name:         kube-proxy
Labels:       ["none"]
Annotations:  ["none"]
Role:
  Kind:  Role
  Name:  kube-proxy
Subjects:
  Kind   Name                                             Namespace
  ----   ----                                             ---------
  Group  system:bootstrappers:kubeadm:default-node-token  

## Obtener los pods como si fuesemos el usuario dev-user
kubectl get pods --as dev-user
Error from server (Forbidden): pods is forbidden: User "dev-user" cannot list resource "pods" in API group "" in the namespace "default"

## Creamos un fichero para establecer el rol y lo aplicamos
nano dev-user_rol_file.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list","create","delete"]

kubectl create -f dev-user_rol_file.yaml 
role.rbac.authorization.k8s.io/developer created

## El proceso depende de un segundo fichero para confirmar y enlazar el rol con el usuario
nano dev-user_rol_binding.yaml
apiVersion: rbac.authentication.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-user-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io

## Si es necesario, se puede crear desde comando
kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
role.rbac.authorization.k8s.io/developer created

## El enlace se puede crear desde comando
kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user
rolebinding.rbac.authorization.k8s.io/dev-user-binding created

## Verificar si el usuario puede realizar una acción
kubectl auth can-i get pods --as dev-user
no

## Mejoramos el fichero para ampliar el acceso
nano dev-user_rol_file.yaml
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    creationTimestamp: "2026-07-17T16:50:45Z"
    name: developer
    namespace: default
    resourceVersion: "2770"
    uid: febc5c1c-9a90-4b67-8230-ef532f0eaa2e
  rules:
  - apiGroups:
    - ""
    resources:
    - pods
    verbs:
    - list
    - create
    - delete
    - get
kind: List
metadata:
  resourceVersion: ""

## El fichero de los enlaces debe ser mejorado
nano dev-user_rol_binding.yaml 
apiVersion: v1
items:
- apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    creationTimestamp: "2026-07-17T16:58:38Z"
    name: dev-user-binding
    namespace: default
    resourceVersion: "3387"
    uid: d813e719-3fee-48a1-aa87-acb7b4a2d030
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: developer
  subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: dev-user
kind: List
metadata:
  resourceVersion: ""

## Si usamos la descripción con formato yaml y exportamos el contenido a un fichero se puede interactuar con sus valores
kubectl describe roles developer -o yaml > file.yaml
kubectl describe rolebings developer -o yaml > file.yaml

## Borrar el role de developer
kubectl delete roles.rbac.authorization.k8s.io developer 
role.rbac.authorization.k8s.io "developer" deleted from default namespace

## Borrar el enlace del role de developer
kubectl delete rolebindings.rbac.authorization.k8s.io dev-user-binding 
rolebinding.rbac.authorization.k8s.io "dev-user-binding" deleted from default namespace

## Volver a aplicar el role & rolebing
kubectl create -f dev-user_rol_file.yaml ; kubectl create -f dev-user_rol_binding.yaml 
role.rbac.authorization.k8s.io/developer created
rolebinding.rbac.authorization.k8s.io/dev-user-binding created

## Editamos el fichero para agregar un resourceName nuevo
kubectl edit role developer -n blue
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2026-07-17T16:29:54Z"
  name: developer
  namespace: blue
  resourceVersion: "5199"
  uid: 3b5f0fd2-464f-46e8-a0f2-e88edea1b405
rules:
- apiGroups:
  - ""
  resourceNames:
  - blue-app
  - dark-blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
role.rbac.authorization.k8s.io/developer edited

## Editamos para agregar un grupo a pie de página y permitir el uso de deployments
kubectl edit role developer -n blue
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2026-07-17T16:29:54Z"
  name: developer
  namespace: blue
  resourceVersion: "5375"
  uid: 3b5f0fd2-464f-46e8-a0f2-e88edea1b405
rules:
- apiGroups:
  - ""
  resourceNames:
  - blue-app
  - dark-blue-app
  resources:
  - pods
  verbs:
  - get
  - watch
  - create
  - delete
- apiGroups:
  - apps
  resources:
  - deployments
  verbs:
    - create

role.rbac.authorization.k8s.io/developer edited