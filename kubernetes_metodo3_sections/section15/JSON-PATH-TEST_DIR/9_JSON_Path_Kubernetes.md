## 123.1º El contenido de los ficheros yaml y las consultas se pueden almacenar e interpretar como JSON, el objetivo de usar JSON es cuando hay muchos componentes que consultar en los despliegues. Estas técnicas también son válidas con la consulta de los componentes de Kubernetes. Cuando se usa el ["kubectl_get_nodes_-o_wide"] ofrece algunos detalles pero no está todo desglosado, si estraemos el código fuente con formato JSON/YAML obtenemos todos los elementos que se puedan consultar. La opción describe también muestra los detalles pero si queremos mostralo como un informe de tablas necesitamos usar el modo JSON_PATH

## 123.2º Indexando en las claves de cada nivel de profundidad obtenemos los valores. En la consulta de nodos, pods, servicios, despliegues, la extración del yaml puede cambiarse por extraer el JSON, el comando de consultas muestra en pantalla una tabla con el contenido del yaml/JSON. Aunque el ["-o wide"] muestra más detalles, sigue siendo incompleto para una consulta profunda. 

## 123.3º Algunos valores ausentes en la salida del comando son ["Cantidad_de_recursos","Condiciones_sobre_nodos","Arquitectura_Hardware","Imagen_usada"]. 

## 123.4º Si queremos desarrollar un informe con forma de tabla con detalles que no se muestran en el kubectl get/describe. Necesitamos usar el JSON_Path. 

## 123.5º Necesitamos 4 conceptos para desarrollar el informe
- 1. Identificar el comando de consulta   --> [kubectl get *]
- 2. Establecer la salida de formato JSON --> [-o JSON]
- 3. Formular la consulta de JSON_PATH    --> [.items[0].spec.containers[0].image]
- 4. Realizar el comando completo --> [kubectl get pods -o=jsonpath={'.items[0].spec.containers[0].image'}]

## 123.6º En las líneas del intérprete de JSON la expresión ["\n"] es para crear lineas nuevas y ["\t"] para tabular los espacios
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' --> master node01
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.architecture}' --> amd64 amd64
kubectl get nodes -o=jsonpath='{.items[*].status.capacity.cpu}' --> 4 4
kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {.items[*].status.capacity.cpu}' --> master node01 4 4
kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\n"} {.items[*].status.capacity.cpu}' --> master node01 "\n" 4 4
kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\t"} {.items[*].status.capacity.cpu}' --> master node01 "\t" 4 4
------------------------------------------------------------------------------------------------------------------------
for each node "\n" print node name "\t" print cpu count "\n" end for
'{range .items[*]} "\n" {.metadata.name} {"\t"} {.status} {"\n"}'
-------------------------------------------------------------------------------------------------------
kubectl get nodes -o=jsonpath='{range.items[*]} {.metadata.name} {"\t"} {.items[*].status.capacity.cpu} {"\n"} {end}'
-----------------------------------------------------------------------------------------------------------------------
kubectl get nodes -o=custom-columns=NODE:.metadata.name,CPU:.status.capacity.cpu
kubectl get nodes --sort-by=.metadata.name
kubectl get nodes --sort-by=.status.capacity.cpu

## 123.7º Para obtener el nodo que está en uso --> kubectl get nodes -o=jsonpath'{.items[*].metadata.name}'

## 123.8º Para obtener la architectura del sistema --> kubectl get nodes -o=jsonpath'{.items[*].status.nodeInfo.architecture}'

## 123.9º Para recibir más valores en un solo comando --> kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{.items[*].status.capacity.cpu}'

## 123.10º Si usamos las expresiones "\n" y "\t" se puede dar formatos de nueva linea y tabulación:
kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\n"} {.items[*].status.capacity.cpu}'
kubectl get nodes -o=jsonpath='{.items[*].metadata.name} {"\t"} {.items[*].status.capacity.cpu}'
