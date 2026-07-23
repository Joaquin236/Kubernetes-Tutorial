## 81.1º Cuando se va a realziar un volumen perisitente, debemos realizar el acceso al recurso cloud primero. Este paso se realiza con un comando para crear el espacio de acceso generando un aprovisionamiento estático:
gcloud beta copute disk create --size 1GB --region us-east1 pd-disk

## 81.2º Si se automatiza obtenemos un aprovicionamiento dinámico con clases de almacenamiento. Google ofrece la opción del aprovisionamiento dinámico y adjuntarlo a los pods que lo soliciten, cuando se crea la clase de almacenamiento, el pod y el reclamador del volumen deben coincidir en el parámetro de enlace ["(POD_file)persistentVolumeClaim.claimName"_With_"(PV_file)metada.name"-->El_valor_de_la_linea_del_fichero_deben_coincidir]

## 81.3º En los ficheros de clase de almacenamiento en la sección de parámetros, se asigna el tipo y la replicación.

## 81.4º En el nombre de la clase de almacenamiento se le puede nombrar por el tipo de tarifa contratada para filtrar por los privilegios de acceso

## 81.5º Muestra de un volumen persistente en yaml
nano pv-definition.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  accessModes:
  - ReadWriteOnce # --> ["ReadWriteMany","ReadOnlyMany","ReadWriteOnce"]
  capacity:
    storage: 500Mi
  gcePersistentDisk:
    pdName: pd-disk
    fsType: ext4
kubectl apply -f pv-definition.yaml
kubectl get persistentvolume

## 81.6º Muestra de fichero de pod con ruta de montaje de volumen
nano Pod_mount_vol.yaml
apiVersion: v1
kind: Pod
metadata:
  name: random-number-generator
spec:
  containers:
  - image: alpine
    name: alpine
    command: ["/bin/sh","-c"]
    args: ["shuf -i 0-100 -n 1 >> /opt/number.out;"]
    volumeMounts:
    - mountPath: /opt
      name: data-volume
  volumes:
  - name: data-volume
    persistentVolumeClaim:
     claimName: myclaim
kubetctl apply -f Pod_mount_vol.yaml

## 81.7º Muestra de fichero yaml para reclamar
nano pvc-definition.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  resources:
   requests:
     storage: 500Mi
kubectl create -f pv-definition.yaml
kubectl get persistentvolumeclaim

## 81.8º Muestra de la clase de almacenamiento
nano sc-definition.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd

## 81.9 Muestra de fichero yaml para reclamar mejorado
nano pvc-definition.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: google-storage
  resources:
   requests:
     storage: 500Mi
kubectl create -f pv-definition.yaml
kubectl get persistentvolumeclaim

## 81.10 Otros parámetros útiles para la clase de almacenamiento, son los parámetros ["type","replication-type"]. Los valores son ["pd-standard_&_pd-ssd","none_&_regional-pd"]. Los plugins compatibles son: ["AWSElasticBlockStore","AzureFile","AzureDisk","CephFS","Cinder","FC","FlexVolume","Flocker","GCEPersistentDisk","Glusterfs","iSCSI","QuoByte","NFS","RBD","VsphereVolume","PortworxVolume","ScaleIO","StorageOS","local"]

## 81.11º Muestra de la clase de almacenamiento mejorado con las opciones de solicitar el medio de almacenamiento
nano sc-definition.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard ["pd-standard","pd-ssd"]
  replication-type: none ["none","regional-pd"]

## 81.12º Las clases de almacenamiento se pueden clasificar y establecer un rango de tarifas para facilitar el filtro de características que ofrecen al cliente, si el cliente asciende y paga más por el servicio se le otorga una clase de almacenamiento mayor

## 81.13º Modelo Plata del fichero de clases
nano sc-definition_silver.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: silver
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-standard 
  replication-type: none

## 81.14º Modelo Oro del fichero de clases
nano sc-definition_gold.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gold
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: none

## 81.15º Modelo Platino del fichero de clases
nano sc-definition_platinum.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: platinum
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: regional-pd