## 70.1º El sistema de docker establece que el kernel del sistema anfitrión es compartido con el contenedor, el sistema operativo tiene su propio espacio de nombres y el contenedor lleva el suyo independiente. Cuando está activo un contenedor, el identificador del proceso tendrá un número distinto al comando normal realizado por el host. Este proceso lo suele ejecutar el usuario root que tiene acceso a cualquier directorio/fichero.* que esté en el sistema, dentro de comando de docker se puede establecer el nombre de usuario.
docker run --user=stand_user ubuntu sleep 2007 # --> iniciar el contenedor con un usuario sin privilegios
ps aux # --> mostrar los procesos en ejecución
docker build -t my-ubuntu-image # --> construir un contenedor con los valores iniciales
docker run my-ubuntu-image sleep 2007 # --> iniciar el contenedor recién construido
ps aux # --> mostrar los procesos activos

## 70.2º El usuario ["root"] que está alojado dentro del contenedor es diferente al usuario ["root"] del sistema anfitrión, mitigando el peligro de recibir más privilegios de los necesarios en el host.

## 70.3º Las capacidades del usuario del contenedor pueden modificarse añadiendo parámetros --cap-add y el módulo a limitar:
docker run --cap-add MAC_ADMIN ubuntu # --> añade una capacidad
docker run --cap-drop KILL ubuntu # --> borra una capacidad
docker run --privileged ubuntu # --> ofrece todos los accesos
