## 102.1º Podemos elegir la versión de algún despliegue añadiendo una flag con la versión que necesitemos
helm install nginx-release bitnami/nginx --version 7.1.0
## 102.2º Si consultamos los pods, debe aparecer el despliegue realizado por el helm
kubectl get pods -o wide --> nginx-release

## Si desglosamos la descripción encontramos más información
kubectl describe pod nginx-release

## Si necesitamos actualizarlo para recibir nuevos cambios, le realizamos el proceso de upgrade
helm upgrade nginx-release bitnami/nginx 

## Volvemos a consultar los pods
kubectl get pods -o wide --> nginx-release

## Volvemos a consultar la descripción del pod
kubectl describe pod nginx-release

## En el listado del helm tiene que aparecer como desplegado y con la revisión 2
helm list

## Para mostrar el historial de cambios se usa el comando
helm history nginx-release

## Para deshacer un cambio de versión y regresar a una revisión atrás usamos
helm rollback nginx-release 1

## 102.3º El sistema necesita permisos para algunas acciones y al realizarlos sin los permisos devuelve errores en la actualización del despliegue. Solicitará una flag con la contraseña del usuario desplegado
helm upgrade wordpress-release bitnami/wordpress

## 102.4º Durante los rollback, los componentes desplegados también son restaurados y con la versión anterior a la restauración, pero los registros de las tablas de las bases de datos siguen sin cambiar