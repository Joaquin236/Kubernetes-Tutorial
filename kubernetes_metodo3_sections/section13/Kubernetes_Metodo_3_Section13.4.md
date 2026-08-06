## 108.1º El comando ["kustomize build k8s/"] no es capaz de desplegar la estructura solicitada solo la crea para establecerla después, para aplicarla y usarla necesitamos añadir la tubería ["|"], detrás de la tubería se inserta el ["kubectl apply -f -"] para completar el proceso
Comando completo --> kustomize build k8s/ | kubectl apply -f -
Otro comando válido es --> kubectl apply -k k8s/

## 108.2º Para borrar un despliegue desde kustomize usamos
kustomize build k8s/ | kubectl delete -f -
kubectl delete -k k8s/

## 108.3º Muestra del fichero kustomization.yaml
nano kustomization_1.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
# kubernetes resources to be managed by kustomize
resources:
  - nginx-depl.yaml
  - nginx-service.yaml
# customizations that need to be made
commonLabels:
  comany: KodeKloud

## 109.1º En algunas ocasiones la estructura del k8s suele ser un directorio y dentro todos los ficheros, cuando la estructura se realiza con más de dos directorios se añade ["k8s/api/"] donde esté ubicado los ficheros del despliegue de la apliación

|Estructura con un dir                    |Estructura con más dir  |
|-----------------------------------------|------------------------|
|k8s                                      |k8s                     |
│├───File1.yaml                           |├───api                 |
│├───File2.yaml                           ||   ├───API_File1.yaml  |
│├───File3.yaml                           ||   └───API_File2.yaml  |
│├───File4.yaml                           |├───db                  |
│├───File5.yaml                           ||   ├───DB_File1.yaml   |
│├───File6.yaml                           ||   └───DB_File2.yaml   |
│├───File7.yaml                           ||                       |
│└───File8.yaml                           ||                       |
|-----------------------------------------|------------------------|
| kubeclt apply -f k8s                    |kubectl apply -f k8s/api|
|kustomize build k8s/ | kubectl apply -f -|kubeclt apply -f k8s/db |

## 109.2º Después de los comandos, se añade el fichero kustomization.yaml (Manualmente) en la raiz del k8s, dentro hay parámetros para configurar

k8s     
├───Kustomization.yaml              
├───api               
|   ├───API_File1.yaml --> api/api-depl.yaml
|   └───API_File2.yaml --> api/api-service.yaml
├───db                
|   ├───DB_File1.yaml  --> api/db-depl.yaml
|   └───DB_File2.yaml  --> api/db-service.yaml
|                     
|                     
## Contenido del fichero genenerado manualmente
nano kustomization_2.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
# kubernetes resources to be managed by kustomize
resources:
  - api/api-depl.yaml
  - api/api-service.yaml
  - db/db-depl.yaml
  - db/db-service.yaml

## 109.3º Otra posible estructura es: Todos los ficheros están dentro de los directorios ["api","db","cache","kafka"]
k8s           
├───api
├───db
├───cache
└───kafka

nano kustomization_3.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
# kubernetes resources to be managed by kustomize
resources:
  - api/api-depl.yaml
  - api/api-service.yaml
  - db/db-depl.yaml
  - db/db-service.yaml
  - cache/config-depl.yaml
  - cache/config-service.yaml
  - cache/config-config.yaml
  - kafka/kafka-depl.yaml
  - kafka/kafka-service.yaml
  - kafka/kafka-config.yaml

## Distribución del fichero kustomization por cada subdirectorio
k8s
├───kustomization.yaml --> resources.["api","db","cache","kafka"]
├───api
|   └───kustomization.yaml
├───db
|   └───kustomization.yaml --> k8s/db/kustomization.yaml --> resources.["-db-depl.yaml","-db-service.yaml"]
├───cache
|   └───kustomization.yaml
└───kafka
    └───kustomization.yaml

## 109.4º Cuando usamos este modo el contenido de los ficheros se unifica para compilar una estructura yaml de despliegue, el comando solo crea una vista previa en la consola, para aplicarla se añade el comando secundario de aplicar. El fichero kustomization.yaml se puede repetir dentro de los subdirectorios

