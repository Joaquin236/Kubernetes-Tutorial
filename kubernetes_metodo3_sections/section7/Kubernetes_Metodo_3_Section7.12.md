## 72.1º El tráfico de la conexión tiene diversos puertos. El usuario se conecta a través del puerto 80, la aplicación se conecta a la api desde el puerto 5000 y la api conecta con el servidor desde el puerto 3306. El usuario recibe este contenido cuando las conexiones son exitosas.

## 72.2º Hay dos typos de tráfico: ["ENTRADA","SALIDA"]

## 72.3º En los servidores web, el usuario es la entrada, cuando envía la información solicitada a la api es la salida. La api recibe la petición y la procesa para solicitar el contenido de la base de datos

## 72.4º La estructura del tráfico se mostraría así:
|Nº de pasos|Tipo de tráfico|Puerto|Componente   |
|-----------|---------------|------|-------------|
|   1       |   Entrada     |  80  |Servidor web |
|   2       |   Salida      | 5000 |Servidor web |
|   3       |   Entrada     | 5000 |API          |
|   4       |   Salida      | 3306 |API          |
|   5       |   Entrada     | 3306 |Base de datos|

## 72.5º En cada cluster, nodo, pod y contenedor hay direcciones IP para las conexiones y acceder, cada enlace lleva una ruta de enrutamiento, por defecto está el acceso completo a la red sin recticciones.

## 72.6º Los servicios al estar conectados entre sí, un servicio puede tener acceso directo, si una organización exige que esto no suceda necesitamos las políticas de red. El tráfico de red puede filtrarse a través de políticas de red, encapsulando el servicio y solo lo puede acceder de forma indirecta. Solo se aplica al pod que tiene la política activada, las opciones se aplican con las etiquetas de label.role y podSelector.matchLabels.role de los ficheros yaml de las replicas.

## 72.7º Muestra de la estructura de politica de red
policyTipes:
- Ingress # --> Allow Ingress
ingress:
- from: 
  - podSelector:
      matchLabels:
        name: api-pod # --> from API Pod
  ports:
  - protocol: TCP
    port: 3306 # --> Port 3306

## 72.8º Muestra de la estructura de roles del pod
podSelector:
  matchLabels:
    role: db

## 72.9º La estructura de una politica de red completa se establece así;
nano network_policy-definition.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTipes:
  - Ingress # --> Allow Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod # --> from API Pod
    ports:
    - protocol: TCP
      port: 3306 # --> Port 3306
kubectl create -f network_policy-definition.yaml

## 72.10º Las políticas de red son aplicadas por la solución de red implementada en el cluster. No todas las soluciones de red son compatibles con la politica de red. Las herramientas de soporte de politica de red son: ["Kube-Router","Calico","Romana"]. Si la infraestuctura está conectada a la aplicación ["Flannel"] no se aplicará las politicas de red del cluster. La documentación de politica de red ofrece los manuales de como configurar las estructura necesarias. Si un cluster no admite estas politicas se pueden crear pero no aplicar ni mostrará mensaje de error.