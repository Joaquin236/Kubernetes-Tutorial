## 75.1º Cuando realizamos un despliegue, creamos un recurso, lo editamos o borramos, algunas acciones son realizadas por el controlador. El controlador es un proceso en segundo plano para tomar los parámetros y realizar las acciones necesarias del funcionamiento de kubernetes.

## 75.2º Muestra del código del controlador de despliegues en lenguaje GO. (Este modelo no es funcional por las líneas ausentes: ["CODE_HIDDEN_DETECTED"])
// DEBUG: HEADER PACK AND VARIABLE
package deployment
var controllerKind=apps.SchemeGroupVersion.Withkind("Deployment")
// WARNING: CODE HIDDEN
// DEBUG: RUN BEGINGS WATCHING AND SYNCING
func(dc *DeploymentController) Run(workers int, stopCh <-chan struct{})
// WARNING: CODE HIDDEN
// DEBUG: ADD REPLICASET
func(dc *DeploymentController) addReplicaSet(obj interface{})
// WARNING: A LOT OF CODE HIDDEN

## 75.3º Los recursos conectan con el controlador a través del etcd:
|Resources  |Etcd Component |Controller |
|-----------|---------------|-----------|
|ReplicaSet |<------------->|ReplicaSet |
|Deployment |<------------->|Deployment |
|Job        |<------------->|Job        |
|CronJob    |<------------->|CronJob    |
|Statefulset|<------------->|Statefulset|
|Namespace  |<------------->|Namespace  |

## 75.4º Cada recurso que se vaya a crear tiene que coincidir con una api que sea válida para ser creado, cuando no coincide se muestra un error indicando que no hay coincidencias con la cabecera establecida.

## 75.5º Existe una opción de crear una definición de recursos personalizados para ofrecer una cabecera válida para crear los recursos que necesitemos sin sufrir el error de ["cabecera_no_encontrada"]. El fichero para declarar la cabecera personalizada es un fichero extra-largo.