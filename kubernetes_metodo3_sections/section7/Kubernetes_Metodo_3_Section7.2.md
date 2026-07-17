## 62.1º Una vez configurado o reparado los accesos a los cerfificados, nuestra cuenta de administrador puede seguir con sus funciones. Si un trabajador nuevo de incorpora para elegar parte del proyecto, necesitmos crear una cuenta nueva para el nuevo puesto. La cuenta nueva debe pertenecer al grupo de Administación del proyecto ["admin"]. El certificado de la cuenta nueva debe firmarse y enviarse al servidor, al validar la autenticación puede acceder al cluster. Al caducar el fichero se genera otro nuevo con la fecha renovada.

## 62.2º El servicio Autoridad Certificativa se encarga de administrar los datos de los certificados y solo este componente puede hacer esta tarea. El servidor maestro/Master-Node es también el servidor de los CA. Cuando crece el número de usuarios de la estructura de kubernetes, también crece el uso de los certificados, el certificade_API se encarga de esta sección.

## 62.3º El administrador incia los procecos de interacción con la API del CA.
## 1º --> Crear el objeto de la respuesta de la firma
## 2º --> Revisar la respuesta
## 3º --> Aprobar la respuesta
## 4º --> Compartir los certificados con los ususarios

## 62.4º El adminsitrador debe usar los comandos para administrar cada certificado: openssl genrsa -out ["user.key"] 2048 --> crea la clave privada openssl req -new -key ["user.key"] -subj ["/CN=user"] -out ["user.csr"] 

## 62.5º  El administrador toma el fichero.csr creado y lo asigna a un fichero.yaml
nano user-cr.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metada:
  name: ["user-csr"]
spec:
  signerName: Kubernetes.io/kube-apiserver-client
  expirationSeconds: 9999 # --> seconds for expiry
  usages:
  - client auth
  request: <Base64_ENCODED_CSR>

## 62.6º Después recolecta el fichero csr y convierte su contenido en base64:
cat user.csr | base 64
["BASE64_CODE"]

## 62.7º Creamos un fichero yaml que lleve el código en su interior
nano certificate_file.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metada:
  name: ["user-csr"]
  creationTimestrap: "2021-08-13T20:30:00Z"
  uid: 3423f-324a-dfd981-198b
spec:
  signerName: Kubernetes.io/kube-apiserver-client
  expirationSeconds: 9999 # --> seconds for expiry
  usages:
  - client auth
  request: ["Base64_ENCODED_CSR"]
  conditions:
  - type: Aproved
    status: "True"
    reason: KubectlAprove
    message: "Commentary"
    lastUpdateTime: "2021-08-13T20:30:00Z"

## 62.8º Mostramos el estado del csr:
kubectl get csr 
Name            AGE     Condition
["name"]        12s     Pending

## 62.9º Aprovar el certificado:
kubectl certificate approve ["name"]

## 62.10º Visualizar un csr con formato:
kubectl get csr ["name"] -o yaml

## 62.11º Decodificar un elemento que esté en base 64
kubectl get csr ["name"] -o yaml | grep -A7 certificate | base64 --decode

## 62.12º El Controller-manager es el encargado de realizar las tareas de administrar los certificados, dentro hay módulos que se encargan de los procesos como csr-approving & csr-signing

## 62.13º Mostrar el fichero kube-controller.yaml
cat /etc/kubernetes/kube-controller-manager.yaml
-cluster-signing-cert-file=/etc/kubernetes/pky/ca.crt
-cluster-signing-key-file=/etc/kubernetes/pky/ca.key 

## Obtener el código base64 de un certificado válido:
cat akshay.csr | base64 -w 0

## Crear un fichero yaml para instalarlo en kubernetes:
nano cretificate_deploy.yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  groups:
  - system:authenticated
  request: <Paste the base64 encoded value of the CSR file>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth

## Aplicar el fichero creado:
kubectl apply -f akshay_yaml_file.yaml 
certificatesigningrequest.certificates.k8s.io/akshay created

## Consultar el estado del certificado:
kubectl get csr
NAME        AGE     SIGNERNAME                                    REQUESTOR                  REQUESTEDDURATION   CONDITION
akshay      2m42s   kubernetes.io/kube-apiserver-client           kubernetes-admin           ["none"]              Pending
csr-jdl7z   27m     kubernetes.io/kube-apiserver-client-kubelet   system:node:controlplane   ["none"]              Approved,Issued

## Aprobar manualmente el certificado:
kubectl certificate approve akshay
certificatesigningrequest.certificates.k8s.io/akshay approved

## Localizar un csr específico:
kubectl get csr | grep agent*
agent-smith   86s     kubernetes.io/kube-apiserver-client           agent-x                    ["none"]              Pending

## Localizar el origen de un csr específico:
kubectl get csr agent-smith -o yaml | grep system
  - system:masters
  - system:authenticated

## Denegar el csr:
kubectl certificate deny agent-smith
certificatesigningrequest.certificates.k8s.io/agent-smith denied

## Borrar el csr denegado:
kubectl delete csr agent-smith 
certificatesigningrequest.certificates.k8s.io "agent-smith" deleted
