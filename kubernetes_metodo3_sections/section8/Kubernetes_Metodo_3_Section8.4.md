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
kind: PersitenVolmeClaim
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

##
kubectl get pods -o wide 
NAME     READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
webapp   1/1     Running   0          17s   172.17.0.4   controlplane   ["none"]           ["none"]

##
kubectl exec webapp -- cat /log/app.log
[2026-07-22 16:23:24,496] INFO in event-simulator: USER1 is viewing page2
[2026-07-22 16:23:25,497] INFO in event-simulator: USER4 logged out
[2026-07-22 16:23:26,498] INFO in event-simulator: USER1 logged out
[2026-07-22 16:23:27,499] INFO in event-simulator: USER4 is viewing page1
[2026-07-22 16:23:28,500] INFO in event-simulator: USER1 is viewing page1

##
kubectl delete pods webapp 
pod "webapp" deleted from default namespace

kubectl get pods -o wide 
No resources found in default namespace.

kubectl exec webapp -- cat /log/app.log
Error from server (NotFound): pods "webapp" not found

##
