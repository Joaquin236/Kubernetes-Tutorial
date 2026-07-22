## 77.1º La estructura del sistema de archivos de Docker se establece así:
/var/lib/docker
├─> aufs
├─> containers
├─> image
└─> volumes

## 77.2º La arquitectura del Docker se ejecuta con el comando docker build Dockerfile, dentro del fichero deben estar las instrucciones para establecer el contenedor que necesitemos establecer:
nano Dockerfile (No dispone de extension de fichero)
FROM Ubuntu
RUN apt update && apt -y install python
RUN pip install flask flask-mysql
COPY . /opt/source-code
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run

docker build DockerFile -t my-custom-app

## 1º Capa: Capa base de Ubuntu
## 2º Capa: Cambios en los paquetes APT
## 3º Capa: Cambios en los paquetes PIP
## 4º Capa: Código Fuente
## 5º Capa: Actualizar Entrypoint
## 6º Capa: Capa del Contenedor

## 77.3º Las 5 primeras capas son ["solo-lectura"], mientras la capa 6 puede tener actos de esctritura. Solo permanece cuando está en ejecución el contenedor, al terminar el proceso la capa 6 se borra. La capa de imagen puede ser compartida por otros contenedores.

|Capa Contenedor/Lectura+Escritura | Capa Imagenes/Solo-Lectura                |
|----------------------------------|-------------------------------------------|
|temp.txt (este fichero se borrará)|app.py (este fichero si permanece)         |
|app.py (esta copia se borrará)    |source_code2.py (este fichero si permanece)|

## 77.4º Los volumenes de archivos de Docker tienen como objetivo mantener los ficheros de la capa de uso temporal y evitar perderlos. Necesitamos crear los puntos de montaje para declararlos en el comando de creación del contenedor.
docker volume create data_volume
/var/lib/docker
├─> volumes
└─> data_volume

## 77.5º Después establecemos un contenedor con el nuevo volumen declarado, elegimos la ruta con los ficheros y nombre del servicio. 
docker run -v /data/mysql:/var/lib/mysql mysql
docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql

## 77.6º Los controladores de almacenamiento de Docker son: ["AUFS","ZFS","BTRFS","Decive_Mapper","Overlay","Overlay2"]. El sistema elige los controladores que vaya a uar.

## 77.8º Los volumenes de montaje no los maneja el controlador de almacenamiento. Lo maneja los plugins de los volumenes. El plugin predeterminado para el volumen es el local. 

## 77.9º El plugin ayuda a crear un volumen en el host de Docker y almacena en el directorio /var/lib/*

## 77.10º También existen aplicaciones de terceros que pueden descargarse y aplicarse en nuestro entorno.

## 77.11º Cuando la unidad del volumen está conectada a la nube, hay que usar un servicio que ofrezca mantener los ficheros incluso después de la desconexión.
docker run -it --name mysql --volume-driver rexray/ebs --mount src=ebs-vol,target=/var/lib/mysql mysql
