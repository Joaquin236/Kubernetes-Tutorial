## 79.1º Los contenedores de Docker fueron diseñados para ser transitorios, después de utilizarlos pierden el valor que se les daba y se destruye después de terminar su uso. Para prevernir la pérdida de datos se les establece el volumen de montaje de directorios permanente.

## 79.2º Al igual que sucede en Docker, los pods de Kubernetes también tienen el diseño de ser transitorios y se borra todos los datos procesados, el pod también puede recibr un volumen de directorio para prevervar su contenido al borrarse. Este sistema de volumenes no es recomendable en un despliegue de varios nodos, los pods van a usar el mismo directorio en cada nodo y se espera que todos ellos sean iguales, como están en servidores diferentes, no son lo mismo, una solución es replicar la estructura de almacenamiento en otro cluster.

## 79.3º Muestra de fichero de pod con ruta de montaje de volumen
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
  hostPath:
    path: /data
    type: Directory
kubetctl apply -f Pod_mount_vol.yaml

## 79.4º Muestra modificada de la sección volumes del fichero yaml
  volumes:
  - name: data-volume
  awsElasticBlockStore:
    volumeID: ["volume-id"]
    fsType: ext4
