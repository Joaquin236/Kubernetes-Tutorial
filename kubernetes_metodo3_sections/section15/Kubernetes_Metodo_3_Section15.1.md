## 119.1º El contenido de los ficheros yaml y las consultas se pueden almacenar e interpretar como JSON, el objetivo de usar JSON es cuando hay muchos componentes que consultar en los despliegues.

## 119.2º Indexando en las claves de cada nivel de profundidad obtenemos los valores. En la consulta de nodos, pods, servicios, despliegues, la extración del yaml puede cambiarse por extraer el JSON, el comando de consultas muestra en pantalla una tabla con el contenido del yaml/JSON. Aunque el ["-o wide"] muestra más detalles, sigue siendo incompleto para una consulta profunda. 

## 119.3º Algunos valores ausentes en la salida del comando son ["Cantidad_de_recursos","Condiciones_sobre_nodos","Arquitectura_Hardware","Imagen_usada"]. 

## 119.4º Si queremos desarrollar un informe con forma de tabla con detalles que no se muestran en el kubectl get/describe. Necesitamos usar el JSON_Path. 

## 119.5º Necesitamos 4 conceptos para desarrollar el informe
- 1. Identificar el comando de consulta   --> [kubectl get *]
- 2. Establecer la salida de formato JSON --> [-o JSON]
- 3. Formular la consulta de JSON_PATH    --> [.items[0].spec.containers[0].image]
- 4. Realizar el comando completo --> [kubectl get pods -o=jsonpath={'.items[0].spec.containers[0].image'}

## 119.6º
