## 73.1º El servidor de bases del pod está conectado al servidor web y el servidor de api, el objetivo es bloquear el acceso directo desde el servidor web para que solo lo esté conectado a la api. Cuando se ha permitido una conexión entante, se autoriza automáticamente.

## 73.2º En una politica de red normal puede autorizar otras apis de otros espacios de nombres, si necesitamos filtrar los espacios de nombres añadimos bebajo del podSelector.matchLabels.name, la etiqueta namespaceSelector.matchLabels.kubernetes.io/metadata.name. 

## Muestra_1 de fichero yaml
nano network_policy_1.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    ports:
    - protocol: TCP
      port: 3306

## 73.3º También existe la opción de tener que conectar con un servidor de copiar los ficheros de la base de datos. Si conocemos la dirección IP del servidor, lo establecemos con la etiqueta ipblock.
## Muestra_2 de fichero yaml
nano network_policy_2.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    - ipBlock:
          cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306

## 73.4º Cuando no esté disponible estos datos en el fichero yaml, todos los pods pueden llegar hasta el contenedor de bases de datos, los pods que estén fuera no podrán conectar.
## Muestra_3 de fichero yaml
nano network_policy_3.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress
  - from:
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    - ipBlock:
          cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306

## 73.5º A parte de la etiqueda ingress, se puede declarar la etiqueta egress para establecer las normas de la salida de los metadatos del contenedor.
## Muestra_4 de fichero yaml
nano network_policy_4.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80

## Consultar las politicas de red activas
kubectl get networkpolicies.networking.k8s.io 
NAME             POD-SELECTOR   AGE
payroll-policy   name=payroll   34s

## Consultar la descripción de la politica de red payroll-policy
kubectl describe networkpolicies payroll-policy 
Name:         payroll-policy
Namespace:    default
Created on:   2026-07-21 14:08:04 +0000 UTC
Labels:       ["none"]
Annotations:  ["none"]
Spec:
  PodSelector:     name=payroll
  Allowing ingress traffic:
    To Port: 8080/TCP
    From:
      PodSelector: name=internal
  Not affecting egress traffic
  Policy Types: Ingress

## Consultar los pods y la IP de cada uno
kubectl get pods -o wide 
NAME       READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
external   1/1     Running   0          13m   172.17.0.5   controlplane   ["none"]           ["none"]
internal   1/1     Running   0          13m   172.17.0.6   controlplane   ["none"]           ["none"]
mysql      1/1     Running   0          13m   172.17.0.7   controlplane   ["none"]           ["none"]
payroll    1/1     Running   0          13m   172.17.0.8   controlplane   ["none"]           ["none"]

## Con el menú de acceso, verificar el estado de conexión del pod payroll (Conexión interna)
Internal Facing Application Connection test
Host Name --> 172.17.0.8
Host Port --> 8080
Status    --> Success!

## Con el menú de acceso, verificar el estado de conexión del pod payroll (Conexión externa)
External Facing Application Connectivity Test
Host Name --> 172.17.0.8
Host Port --> 8080
Status    --> timed out

## Con el menú de acceso, verificar el estado de conexión del pod external (Conexión interna)
Internal Facing Application Connectivity Test
Host Name --> 172.17.0.5 
Host Port --> 8080
Status    --> Success!

## Crea un fichero para configurar una politica de salida
nano network_policy_egress_pyaroll.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
   matchLabels:
    name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
   - {}
  egress:
  - to:
    - podSelector:
       matchLabels:
        name: mysql
    ports:
    - protocol: TCP
      port: 3306
  - to:
    - podSelector:
       matchLabels:
        name: payroll
    ports:
    - protocol: TCP
      port: 8080
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP

kubectl apply -f network_policy_egress_pyaroll.yaml 
networkpolicy.networking.k8s.io/internal-policy created
