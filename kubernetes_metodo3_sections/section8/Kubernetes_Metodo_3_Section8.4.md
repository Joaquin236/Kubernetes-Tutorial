## 80.1º El volumen persistente ofrece una centralización de los volumenes de montaje, en el modo normal los pods deben ser modificados uno por uno. Lo mejor de usar el modo persistente es adpatar los pods una vez y se ajusta al servicio de almacenamiento

## 80.2º Muestra de un volumen persistente en yaml
nano pv-definition.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce # --> ["ReadWriteMany","ReadOnlyMany","ReadWriteOnce"]
  capacity:
    storage: 1Gi
  awsElasticBlockStore:
    volumeID: ["volume-id"]
    fsType: ext4
kubectl apply -f pv-definition.yaml
kubectl get persistentvolume

## 80.3º Las reclamaciones de volumen son objetos diferentes frente al objeto de volumen_persistente pero dependen entre sí. El administrador crea el volumen y el usuario crea el objeto de reclamar el volumen. El sistema debe localizar el volumen que lleve la capacidad suficiente, el modo de acceso, el modo de volumen, el tipo de selector y la clase de almacenamiento.

## 80.4º Cuando hay un volumen en uso y otro pod no puede usarlo estará en estado ["pendiente"] hasta que lo pueda recibir. Tras recibirlo empieza a trabajar, los metadatos deben coincidir entre el volumen_persistente y el reclamador_de_volumen

## 80.5º Muestra de fichero yaml para reclamar
nano pv-definition.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclain
spec:
  accessModes:
  - ReadWriteOnce
  resources:
   requests:
     storage: 500Mi
kubectl create -f pv-definition.yaml
kubectl get persistentvolumeclaim

## 80.6º Si necesitamos borrar un volumen persistente que no sea necesario, usamos el comando
kubectl delete persistentvolumeclaim myclaim

## 80.7º Si usamos el parámetro ["persistentVolumePolicy:Retain"] estamos preservando los fichero del volumen, al realizar esta acción estamos bloqueando el acceso a otros pods a usar este recurso, si queremos borrar los ficheros para limpiar el volumen el parámetro ["persistentVolumePolicy:Delete"] lo limpia para ofrecer el recurso a otro pod, otra opción es el reciclado de los ficheros ["persistentVolumePolicy:Recycle"], el modo de reciclado de fichero está obsoleto debido a que el módulo de limpieza realizaba un esfuerzo adicional para limpiar el contenido, a parte que llamaba al comando rm -fr /scrub/*, intentando hacer una limpieza manual de los adminsitradores con un borrado de nivel de fichero, no garantizaba una limpieza segura, no verificaba las instantáneas ni ofrecía metadatos del proveedor y solo funcionaba con algunos plugins compatibles

## 80.8º Muestra de un pod preparado para recibir la reclamación del volumen
nano pod-definition.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
kubectl create -f pod-definition.yaml

## Consultar los pods activos
kubectl get pods -o wide 
NAME     READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
webapp   1/1     Running   0          17s   172.17.0.4   controlplane   ["none"]           ["none"]

## Consultar los log del pod
kubectl exec webapp -- cat /log/app.log
[2026-07-22 16:23:24,496] INFO in event-simulator: USER1 is viewing page2
[2026-07-22 16:23:25,497] INFO in event-simulator: USER4 logged out
[2026-07-22 16:23:26,498] INFO in event-simulator: USER1 logged out
[2026-07-22 16:23:27,499] INFO in event-simulator: USER4 is viewing page1
[2026-07-22 16:23:28,500] INFO in event-simulator: USER1 is viewing page1

## Exportar el pod para editarlo
kubectl get pods webapp -o yaml > webapp_pod_editable.yaml

## Borrar el pod después de exportar
kubectl delete pods webapp 
pod "webapp" deleted from default namespace

## Localizar los pods, si no hay muestra que no se enocntró
kubectl get pods -o wide 
No resources found in default namespace.

## Localizar el log, si se ha borrado no debe encontrarse
kubectl exec webapp -- cat /log/app.log
Error from server (NotFound): pods "webapp" not found

## Edita el fichero yaml para adaptarlo
nano webapp_pod_editable.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume
  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: Directory
kubectl replace -f webapp_pod_editable.yaml --force
pod/webapp replaced

## Crea un nuevo fichero para crear un Volumen Persistente
nano pv-log.yaml 
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
   path: /pv/log
kubectl apply -f pv-log.yaml 
persistentvolume/pv-log created

## Crea un fichero nuevo para crear un Reclamador de Volumenes
nano claim-log1.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
kubectl apply -f claim-log1.yaml 
persistentvolumeclaim/claim-log-1 created

## Cosulta los Reclamadores de Volumen activos
kubectl get persistentvolumeclaims -o wide 
NAME          STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE     VOLUMEMODE
claim-log-1   Pending                                                     ["unset"]                 2m38s   Filesystem

## Consulta los Volumenes Persistentes activos
kubectl get persistentvolume -o wide 
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE     VOLUMEMODE
pv-log   100Mi      RWX            Retain           Available                          ["unset"]                          9m56s   Filesystem

## Edita el Reclamador para adaptarlo a la nueva especificación
nano claim-log1.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 50Mi
kubectl replace -f claim-log1.yaml --force 
persistentvolumeclaim "claim-log-1" deleted from default namespace
persistentvolumeclaim/claim-log-1 replaced      

## Consulta su estado
kubectl get persistentvolumeclaims 
NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
claim-log-1   Bound    pv-log   100Mi      RWX                           ["unset"]                 67s

## Edita el pod para adaptarlo al reclamador de volumen
nano webapp_pod_editable.yaml
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
kubectl replace -f webapp_pod_editable.yaml --force 
pod/webapp replaced      

## Borra el pod Webapp
kubectl delete pod webapp 
pod "webapp" deleted from default namespace

## Consulta el reclamador de volumen
kubectl get persistentvolumeclaims -o wide 
NAME          STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
claim-log-1   Deleted    pv-log   100Mi      RWX                           ["unset"]                 10m   Filesystem

## Consulta el volumen persistente activo
controlplane ~ ➜  kubectl get persistentvolume -o wide 
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE   VOLUMEMODE
pv-log   100Mi      RWX            Retain           Available    default/claim-log-1                  ["unset"]                          22m   Filesystem
