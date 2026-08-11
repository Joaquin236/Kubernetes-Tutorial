## 116.1º Durante las estancias de los clusteres, existe la posibilidad de producirse fallas en el acceso, el cliente puede se afectado por un panel de error ["300","400","500"]. Necesitamos evaluar el entorno con los comandos de describir/logs del cluster, servicios y pods

## 116.2º Es importante diseñar el mapa de la conexión de los servicios del cluster, evaluar la IP y el puerto al que están concetados, el error puede estar en la sección de front-end o back-end, en la base de datos o un fichero alojado en la rutas del almacenamiento que se haya movido, dañado o borrado.

## 116.3º Un posible mapa lineal de la red puede ser este
Usuario <--> Servicio_Web <--> Web <--> Servicio_de_bases_de_datos <--> Motor_de_bases_de_datos

## 116.4º La verificación del cluster se realiza con los comandos
curl http://web-service-ip:node-port # --> cuando el recurso está fuera de servicio muestra un mensaje de tiempo agotado
kubectl describe web
kubectl logs web
kubectl logs web -f
kubectl logs web -f --previus
