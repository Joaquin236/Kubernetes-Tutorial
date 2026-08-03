## 97.1º Comparación entre las versiones del Helm:
- 1.0 -> Lanzado en Febrero 2016
- 2.0 -> Lanzado en Noviembre 2016
- 3.0 -> Lanzado en Noviembre 2019

## 97.2º La versión de helm para trabajar es la 3.0, la herramienta tiene una interfaz interactiva desde la consola de comandos. En la versión 2.0 se basaba en roles, las definiciones se personalizaban. Para realizar las acciones, se instalaba un paquete adicional llamado Tiller que conectaba el Cluster del Kubernetes con el Helm para realizar las peticiones del adminsitrador. A parte de la presencia de un intermediario, estaba las vulnerabilidades por usar varios programas para este propósito. Tiller llevaba privilegios de administración y modo Dios. Algunas de sus acciones pueden llegar al Kernel de las aplicaciones. Cualquiera que alcance el Tiller podrá realizar cambios no deseados y dañar los sistemas.

## 97.3º Cuando se mejoró el módulo de conexiones de Kubernetes y llegó el Helm-3.0 se borró el Tiller para evitar el intermediario. 

## 97.4º Tabla comparativa entre el Tiller y 3-Way
|___________________________|HELM-2.0|HELM-3.0|
|---------------------------|--------|--------|
|Tiller                     |YES     |NO      |
|---------------------------|--------|--------|
|Parche de Estrategia 3-Way |NO      |YES     |

## 97.5º Para adminsitrar wordpress desde helm usamos
helm install wordpress # --> crea la instantanea-1
helm upgrade wordpress # --> crea la instantanea-2
helm rollback wordpress # --> restaura la instantanea

## 97.6º Si adminsitramos el wordpres por otro comando ["kubectl"] puede hacer un descuadre con la administración. No detecta el cambio, el proceso no tiene efecto comparado con el ejemplo anterior.

## 97.7º El Helm-3.0 comparará la instantanea en uso con que queremos revertir y el estado activo. A través del 3-Way Rollback System si puede revertir los cambios.