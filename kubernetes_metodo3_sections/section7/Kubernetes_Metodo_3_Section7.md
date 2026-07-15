## 57.1º Los servicios de kubernetes deben ser protegidos en caso de que entre el servidor y el cliente hay un atacante de hombre en medio (man in the middle ["Cracker"]), los métodos de protección suele ser a través de cifrar con certificado y claves.

## 57.2º La autenticación determina quíen puede obtener el acceso: ["Static_Token_File","Certificates","External_Auth_Providers-LDAP","Service_Accounts"].

## 57.3º La autorización implementa controles de acceso basados en roles, el usuario se asocia en grupos con permisos restrictivos: ["RBAC_Auth","Node_Auth","ABAC_Auth","Webhook_Mode"]

## 57.4º El certificado TLS se establece para blindar los servicios del kube-apiserver, el kube-proxy, kube-scheduler, kubelet, etcd_cluster and kube_Controller.

## 57.5º La politica de red determina como se conectará los pods a cada cluster y lo que se pueda realizar conectado.

## 57.6º El cluster está atendido por el administrador/operador encargado del despliegue y por el desarrollador que construye la aplicación, evalúa como se desarrollará las actualizaciones de la aplicación y corrige las incidencias de la aplicación. El cliente se conecta para recibir la información obtenida del contenedor que ofrece sus datos, los bots automatizan algunos ajustes de la infraestructura. Los nodos, deploy y pods están conectados y dependen de autorizaciones y autenticaciones para enlazarse.

## 57.7º Los comandos para crear y consultar cuentas son:
kubectl create serviceaccount sa1
kubectl get serviceaccount

## 57.8º Cada cuenta de acceso es administrada por el sistema de kubernetes en lugar del sistema operativo. Las peticiones son interpretadas por kubernetes_api-server.

## 57.9º El kube-apiserver se autentica con certificados, el token_file y servicios de cuenta.

## 57.10º El sistema de autenticación guarda un fichero.csv con la información de los usuarios y grupos. La contraseña se guarda con tokens de codificación.

## 57.11º No se recomienda guardar los usuarios, contraseñnas y grupos en ficheros de texto plano que pueden ser interceptados por entidades ajenas al servicio.

## 57.12º El volumen de montaje debe pasar por el kubeadm setup.

## 57.13º Configura una autorización basada en roles para los usuarios nuevos.

## 58.1º Si el cliente entra en una conexión no protegida por cifrado TLS cuando sea interceptado por el Cracker, el craker se llevará los datos en texto plano y tomará el control de la cuenta del cliente, una de las soluciones es; el contenido se cifra con un certificado y clave privada, el servidor recibe la clave pública, si la clave es interceptada el contenido puede ser descrifrado por el cracker.

## 58.2º El cifrado asimétrico SSH es útil cuando no se ofrece el servicio de autenticación por usuario/contraseña, se puede usar a través de los comandos:
ssh-keygen
cat ~/.ssh/autorized_keys
ssh -i id_ras user1@server1
openssl genrsa -out my-bank.key 1024
openssl rsa -in my-bank.key -pubout > mybank.pem

## 58.3º Si el atacante aún está interesado en capturar la cuenta del usuario puede recrear la página web con el servicio. Si el usuario no percibe que la página que está usando es una web falsa, continuará con el proceso de iniciar sesión, aunque esté el contenido cifrado, el atacante recibe las claves públicas para descubrir las credenciales y tomar el control de la cuenta, el cliente puede no ser consciente de la situación o cuando lo descubre la cuenta de Crakeada. El navegador también puede evaluar el estado del certificado y si no es seguro, da el aviso de servicio no seguro, cada certificado está firmado por una autoridad certificativa y también las empresas de seguridad informática evaluan los certficidados. Las claves públicas llevan la extenxión: ["*.crt","*.pem"]. Las claves privadas ["*.key","*-key.pem"]. Los certificados llevan ["*.crt"].

## 59.1º El administrador debe establecer una conexión segura entre los componentes del cluster, el servidor interno, el planificador y otros nodos. El servidor lleva su certificado con su clave privada, cada componente del servidor lleva un grupo de certificados y claves.

## 59.2º El cliente también lleva sus certificados y clave privada, establece una conexión segura con el proxy del sistema, el etcd server, kubelet, el controller manager y planificador.

## 59.3º Se puede admitir tener más de una entidad certificadora.

## 60.1º Para crear el TSL para usarlo en kubernetes necesitamos el comando:
openssl genrsa -out ca.key 2048 # --> Generar la clave
openssl req -new -key ca.key -sub "/CN=KUBERNETES-CA" -out ca.csr # --> Firmar el certificado de respuesta
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt # --> Firmar el certificado

## 60.2º El usuario admin tiene que realizar el proceso de generar la clave, después generará la firma de certificado de respuesta y el certificado final. Esta forma de identificarse es más segura que los usuarios y contraseñas. Cada componente lleva el suyo con el formato system_user:kubernetes_component_server. Si realizamos 'curl htts://kube-apiserver:6443/api/v1 --key admin.key --cert admin.crt --cacert ca.crt. La consola debe mostrar el fichero JSON del pod en ejecución.

## 60.3º Otra forma de realizar esta acción es creando un fichero yaml donde guardar los parámetros del certificado.
nano kube-config.yaml
apiVersion: v1  
clusters:
- cluster:
    certificate-autority: ca.crt
    server: https://kube-apiserver:6443
  name: kubernetes
kind: Config
users:
- name: kubernetes-admin
  user:
    client_certificate: admin.crt
    client_key: admin.key

## 60.4º El etcd_server es desplegado como un cluster desde múltiples servidores y entorno de alta disponibilidad. Necesita crear certificados adicionales. Con el comando 'cat /etc/kubernetes/manifest/etcd.yaml' 
mostramos los valores de los parámetros del sistema,
entre ellos hay valores para los certificados. Los clientes también necesita disponer de cerificados válidos para el acceso. Los nodos deben estar autenticados y certificados durante la conexión.

## 60.5º El apiserver de kubernetes se puede declarar el certificado con un fihcero openssl.cnf
nano openssl.cnf
[req]
req_extensions=v3_req
distinguished_name=req_distinguished_name
[v3_req]
basicConstrains=CA:FALSE
keyUsage=nonRepudiation,
subjectAltName=@alt_names
[alt_names]
DNS.1=kubernetes
DNS.2=kubernetes.default
DNS.3=kubernetes.default.svc
DNS.4=kubernetes.default.svc.cluster.local
IP.1=10.96.0.1
IP.2=172.17.0.87

## 60.6º El fichero kubelet-config.yaml contiene las rutas de los cetificados:
nano kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/run/systemd/resolve/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/kubelet-node01.crt"
tlsPrivateKeyFile: "/var/lib/kubelet/kubelet-node01.key"

## 61.1º Cuando entramos en un nuevo equipo de desarrollo de kubernetes
# empezamos como administrador del cluster a trabajar.
# Si tuviesemos que desplegar un servicio completo desde cero,
# necesitamos realizar los pasos para crear los certificados y 
# autorizaciones de los grupo para cada usuario.
# Verificar los ficheros:
cat /etc/kubernetes/manifest/kube-apiserver.service
cat /etc/kubernetes/manifest/kube-apiserver.yaml
cat /etc/kubernetes/pki/apiserver.crt


## 61.2º Con el comando ssl y el fichero /etc/kubernetes/pki/apiserver.crt
# obtenemos más información sobre el certificado apiserver.crt:
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
# Después debe aparecer un yaml con datos del apiserver

## 61.3º Tabla de la estrucitura de los certificados de kubernetes:
+--------------------------------------------------+-------------------------------+---------------+----------------+------------+
| Rutas de certificado                             | Nombre del CN                 | Nombre del ALT| Organización   | Issuer     |
| /etc/kubernetes/pki/apiserver.crt                | kube-apiserver                | DNS & IP      |                | kubernetes |
| /etc/kubernetes/pki/apiserver.key                |                               |               |                |            |
| /etc/kubernetes/pki/ca.crt                       | kubernetes                    |               |                | kubernetes |
| /etc/kubernetes/pki/apiserver-kubelet-client.crt | kube-apiserver-kubelet-client |               | system:masters | kubernetes |
| /etc/kubernetes/pki/apiserver-kubelet-client.key |                               |               |                |            |
| /etc/kubernetes/pki/apiserver-etcd-client.crt    | kube-apiserver-kubelet-client |               | system:masters | self       |
| /etc/kubernetes/pki/apiserver-etcd-client.key    |                               |               |                |            |
| /etc/kubernetes/pki/etcd/ca.crt                  | kubernetes                    |               |                | kubernetes |
+--------------------------------------------------+-------------------------------+---------------+----------------+------------+

# 61.4º Kubernetes registra eventos de los procesos y fallos detectados,
# en estos eventos se encuentran las versiones del sistema, direcciones IP, 
# el acceso a los certificados, las autorizaciones y la autenticación. 
# Los comandos son:
journalctl -u etcd.service -l
kubectl logs etcd-master
kubectl logs
# Si el sistema kubernetes se desconecta y necesitamos el panel de eventos, 
# necesitamos usar los comandos de docker:
crictl ps -a
crictl logs <"Container_name">

## Localizar el certificado apiserver.crt del fichero kube-apiserver.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep apiserver.crt
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt

## Localizar el certificado apiserver-etcd-client.crt
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep client
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt

## Localizar la clave apiserver-kubelet-client.key del fichero kube-apiserver.yaml
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep kubelet
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key

## Localizar el certificado server.crt del fichero etcd.yaml
cat /etc/kubernetes/manifests/etcd.yaml | grep server
    - --cert-file=/etc/kubernetes/pki/etcd/server.crt
    - --key-file=/etc/kubernetes/pki/etcd/server.key

## Localizar el fichero ca.crt de la ruta /etc/kubernetes/manifests/etcd.yaml
cat /etc/kubernetes/manifests/etcd.yaml | grep ca  
    - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt

## Localizar la linea Subject del fichero apiserver.crt usando openssl
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep CN
        Issuer: CN = kubernetes
        Subject: CN = kube-apiserver --> It's the answer

## Localizar la linea Issuer del fichero apiserver.crt usando openssl
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep CN
        Issuer: CN = kubernetes --> It's the answer
        Subject: CN = kube-apiserver

## ¿Que DNS falta en el fichero apiserver.crt? --> Falta el kube-master, que no está configurado
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout | grep -A3 Alternative
            X509v3 Subject Alternative Name:
                DNS:controlplane, DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc,
                DNS:kubernetes.default.svc.cluster.local,
                IP Address:172.20.0.1, IP Address:10.244.56.61  
                Signature Algorithm: sha256WithRSAEncryption  

## Localizar el controlplane del fichero server.crt usando openssl
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout | grep control
        Subject: CN = controlplane
                DNS:controlplane, DNS:localhost, IP Address:10.244.56.61, IP Address:127.0.0.1, IP Address:0:0:0:0:0:0:0:1

## Verificar la caducidad del fichero apiserver.crt usando openssl
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text | grep -A2 Validity
        Validity
            Not Before: Jul 14 14:52:55 2026 GMT
            Not After : Jul 14 14:57:55 2027 GMT
            It's 1 Year
## Verificar la caducidad del fichero ca.crt usando openssl
openssl x509 -in /etc/kubernetes/pki/ca.crt -text | grep -A2 Validity
        Validity
            Not Before: Jul 14 14:52:55 2026 GMT
            Not After : Jul 11 14:57:55 2036 GMT
            It's 10 Years

## Verificar si el sistema permite consultar los pods
kubectl get pods
Get "https://controlplane:6443/api/v1/namespaces/default/pods?limit=500": net/http: TLS handshake timeout - error from a previous attempt: unexpected EOF

## Listar para localizar el fichero server
ls -lh /etc/kubernetes/pki/etcd/ | grep server
-rw-r--r-- 1 root root 1.2K Jul 14 17:53 server.crt
-rw------- 1 root root 1.7K Jul 14 17:53 server.key

## Verificar la presencia del kube-apiserver
crictl ps -a | grep kube-apiserver
76bced9cced6d       5c6acd67e9cd1                       53 seconds ago
Exited              kube-apiserver                          7
2a407a582e6bf       kube-apiserver-controlplane            kube-system

## Ver registro del contenedor --> 76bced9cced6d  
crictl logs 76bced9cced6d
1 logging.go:55] [core] [Channel #7 SubChannel #9]grpc: addrConn.createTransport failed to connect
to {Addr: "127.0.0.1:2379", ServerName: "127.0.0.1:2379", BalancerAttributes:
{"<%!p(pickfirstleaf.managedByPickfirstKeyType={})>":
"<%!p(bool=true)>" }}. Err: connection error: desc = "transport: Error while dialing: dial tcp 127.0.0.1:2379: connect: connection refused"
F0714 18:21:10.953387       1 instance.go:233] Error creating leases: error creating storage factory: context deadline exceeded

## Verificar la presencia del etcd
crictl ps -a | grep etcd
552128eeccc4f       0a108f7189562       2 minutes ago
Exited               etcd                      8
e70f965f17074       etcd-controlplane       kube-system

## Ver registro de contonedor --> 552128eeccc4f
crictl logs 552128eeccc4f
E0714 18:35:46.703296   27205 log.go:32] "ContainerStatus from runtime service failed" err="rpc error:
code = NotFound desc = an error occurred when try to find container \"552128eeccc4f\": not found" containerID="552128eeccc4f"
FATA[0000] rpc error: code = NotFound desc = an error occurred when try to find container "552128eeccc4f": not found

##
crictl ps -a | grep kube-api
4de0ee7f825fa       5c6acd67e9cd1                      About a minute ago
Running             kube-apiserver                                12
2a407a582e6bf       kube-apiserver-controlplane            kube-system
3df7a4899437f       5c6acd67e9cd1                           6 minutes ago
Exited              kube-apiserver                                11
2a407a582e6bf       kube-apiserver-controlplane            kube-system

##
crictl logs 4de0ee7f825fa
I0714 18:48:26.821948       1 options.go:263] external host was not specified, using 10.244.253.154
I0714 18:48:26.823667       1 server.go:150] Version: v1.35.0
I0714 18:48:26.823682       1 server.go:152] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
W0714 18:48:26.871908       1 logging.go:55] [core] [Channel #2 SubChannel #4]grpc:
addrConn.createTransport failed to connect to {Addr: "127.0.0.1:2379", ServerName: "127.0.0.1:2379",
BalancerAttributes: {"<%!p(pickfirstleaf.managedByPickfirstKeyType={})>": "<%!p(bool=true)>" }}.
Err: connection error: desc = "transport: Error while dialing: dial tcp 127.0.0.1:2379: operation was canceled"

## El sistema falla porque hay una linea defectuosa en el fichero etcd.yaml
cat /etc/kubernetes/manifests/etcd.yaml | grep server
    - --cert-file=/etc/kubernetes/pki/etcd/server-certificate.crt
    - --key-file=/etc/kubernetes/pki/etcd/server.key

## Renombrar la ruta con el editor nano
nano /etc/kubernetes/manifests/etcd.yaml
- --cert-file=/etc/kubernetes/pki/etcd/server-certificate.crt --> server.crt

## Después de correguir el cambio de nombre de certificado ya debe funcionar
kubectl get pods
No resources found in default namespace.

## Hay otro fallo y se ha detenido el kubernetes
kubectl get nodes
The connection to the server controlplane:6443 was refused - did you specify the right host or port?

## Localizar los contenedores del kube-apiserver
crictl ps -a | grep kube-apiserver
96664a8349a25       5c6acd67e9cd1                    2 minutes ago
Exited              kube-apiserver                        5
c043c4a9e09c8       kube-apiserver-controlplane      kube-system

## Verificar el registro del contenedor
crictl logs 96664a8349a25
I0714 18:59:34.792476       1 options.go:263] external host was not specified, using 10.244.253.154
I0714 18:59:34.794047       1 server.go:150] Version: v1.35.0
I0714 18:59:34.794062       1 server.go:152] "Golang settings" GOGC="" GOMAXPROCS="" GOTRACEBACK=""
W0714 18:59:34.852334       1 logging.go:55] [core] [Channel #2 SubChannel #3]grpc: addrConn.createTransport
failed to connect to {Addr: "127.0.0.1:2379", ServerName: "127.0.0.1:2379", BalancerAttributes:
{"<%!p(pickfirstleaf.managedByPickfirstKeyType={})>": "<%!p(bool=true)>" }}.
Err: connection error: desc = "transport: Error while dialing: dial tcp 127.0.0.1:2379: operation was canceled"

## Hay una linea defectuosa de nuevo en el fichero
nano /etc/kubernetes/manifests/kube-apiserver.yaml
- --client-ca-file=/etc/kubernetes/pki/ca.crt

## Renombramos la ruta del fichero
nano /etc/kubernetes/manifests/kube-apiserver.yaml
- --client-ca-file=/etc/kubernetes/pki/etcd/ca.crt

## Después de correguir el cambio de nombre de certificado ya debe funcionar
kubectl get pods
No resources found in default namespace.
