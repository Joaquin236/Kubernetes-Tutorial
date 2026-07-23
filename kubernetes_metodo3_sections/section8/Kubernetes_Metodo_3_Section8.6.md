## Consultar las clases de almacenamiento
kubectl get storageclasses.storage.k8s.io -o wide 
NAME                   PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  8m43s

## El administrador ha creado más elementos. Vuelve a consultar
kubectl get storageclasses.storage.k8s.io -o wide 
NAME                        PROVISIONER                     RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
local-path (default)        rancher.io/local-path           Delete          WaitForFirstConsumer   false                  9m37s
local-storage               kubernetes.io/no-provisioner    Delete          WaitForFirstConsumer   false                  10s
portworx-io-priority-high   kubernetes.io/portworx-volume   Delete          Immediate              false                  10s

## Localiza una clase de almacenamiento que no adminte el modo-dinamico
kubectl describe storageclasses.storage.k8s.io local-storage 
Name:            local-storage
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"local-storage"},"provisioner":"kubernetes.io/no-provisioner","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner # --> This is the answer
Parameters:            ["none"]
AllowVolumeExpansion:  ["unset"]
MountOptions:          ["none"]
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                ["none"]

## Verifica el estado del valor VolumeBindingMode del local-storage
kubectl describe storageclasses.storage.k8s.io local-storage 
Name:            local-storage
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"local-storage"},"provisioner":"kubernetes.io/no-provisioner","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner
Parameters:            ["none"]
AllowVolumeExpansion:  ["unset"]
MountOptions:          ["none"]
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer # --> This is the answer
Events:                ["none"]

## Localiza el provisionador de la clase de alamacenamiento portworx-io-priority-high
kubectl describe storageclasses.storage.k8s.io portworx-io-priority-high
Name:            portworx-io-priority-high
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"portworx-io-priority-high"},"parameters":{"priority_io":"high","repl":"1","snap_interval":"70"},"provisioner":"kubernetes.io/portworx-volume"}

Provisioner:           kubernetes.io/portworx-volume # --> This is the answer
Parameters:            priority_io=high,repl=1,snap_interval=70
AllowVolumeExpansion:  ["unset"]
MountOptions:          ["none"]
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                ["none"]

## Crea el fichero yaml con las caracteristicas necesarias
nano local-pvc_file.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
   requests:
     storage: 500Mi
kubectl create -f local-pvc_file.yaml 
persistentvolumeclaim/local-pvc created

## Consulta el nuevo elemento
kubectl get persistentvolumeclaims -o wide 
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
local-pvc   Pending                                      local-path     ["unset"]                 37s   Filesystem

## Describe el estado del local-pvc
kubectl describe persistentvolumeclaims local-pvc 
Name:          local-pvc
Namespace:     default
StorageClass:  local-path
Status:        Pending
Volume:        
Labels:        ["none"]
Annotations:   ["none"]
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       ["none"]
Events:
  Type    Reason                Age                  From                         Message
  ----    ------                ----                 ----                         -------
  Normal  WaitForFirstConsumer  3s (x24 over 5m34s)  persistentvolume-controller  waiting for first consumer to be created before binding

## Crea un pod para enlazarlo con la clase de almacenamiento
nano nginx_pvc_file.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx                  
  labels:
   name: nginx
spec:
  containers:
  - image: nginx:alpine
    name: nginx       
    volumeMounts:
    - mountPath: /var/www/html
      name: local-persistent-volume
  volumes:
  - name: local-persistent-volume
    persistentVolumeClaim:
     claimName: local-pvc
kubectl create -f nginx_pvc_pod.yaml 
pod/nginx created

## Consulta el pod nuevo
kubectl get pods nginx -o wide 
NAME    READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          36s   10.22.0.10   controlplane   ["none"]           ["none"]

## Consulta el reclamador de volumen
kubectl get persistentvolumeclaims -o wide 
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
local-pvc   Bound    pvc-c096ff6f-da50-4a70-904e-5ef083e83d38   500Mi      RWO            local-path     ["unset"]                 15m   Filesystem

## Crea otra clase de almacenamiento con las caracteristicas solicitadas
nano delayed-volume-sc_file.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
kubectl create -f  delayed-volume-sc_file.yaml 
storageclass.storage.k8s.io/delayed-volume-sc created

## Consulta el nuevo elemento creado
kubectl get storageclasses.storage.k8s.io -o wide delayed-volume-sc 
NAME                PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
delayed-volume-sc   kubernetes.io/no-provisioner   Delete          WaitForFirstConsumer   false                  43s
