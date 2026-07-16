## 63.1º El archivo KubeConfig se localiza en ~/.kube./config, contiene información sobre clusteres, usuarios y contextos. Cada uno de los clusteres registrados se guardan en las lineas del fichero. Puede guardar información sobre un departamento, servicio en la nube o de una organización. Los usuarios registrados y autorizados a interactuar con los  omponentes del sistema, se guardan en la sección de usuarios. Pueden ser los administradores, los desarrolladores y los instaladores del despliegue de servicios. El contento une y relaciona los usuarios con el cluster. El enlace se forma con el formato: cluster_name@user_name.

## 63.2º Los parámetros se pueden declarar en un fichero yaml:
nano kube_config_declare.yaml
apiVersion: v1
kind: Config
current-context: dev-user@google
clusters:
- name: my-kube-playground
- name: development
- name: production
- name: google

contexts:
- name: my-kube-admin@my-kube-playground
- name: dev-user@google
- name: prod-user@production

users:
- name: my-kube-admin
- name: admin
- name: dev-user
- name: prod-user

## 63.3º En la consola de comandos se puede usar:
kubectl config view --> Comando generico
kubectl config view -kubeconfig-my-custom-config --> Comando con parámetros personalizados
kubectl config view use-context prod-user@production --> Comando para visualizar el contexto prod-user@production
kubectl -h --> mostrar la ayuda del comando

## 63.4º En los ficheros de configuración kube-config también se puede declarar los certificados:
nano kube_config_declare2.yaml
apiVersion: v1
kind: Config
clusters:
- name: production 
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://172.17.0.51:6443

contexts:
- name: admin@production
  context:
    cluster: production
    user: admin

users:
- name: admin
  user:
    client-certificate: /etc/kubernetes/pki/admin.crt
    client-key: /etc/kubernetes/pki/admin.key


## 63.5º También se puede convertir el valor del certificado en base64 y adjuntar el valor, todo tiene que estar en una sola linea:
cat /etc/kubernetes/pki/admin.crt | base64 -o w # --> mostrar en consola el valor con base64

Dentro del fichero debe tener estas lineas y sus valores:
certificate-authority: <El código debe depositarse en una sola linea>
client-certificate: <El código debe depositarse en una sola linea>

Si recolectamos el código y usamos base64 --decode
cat /etc/kubernetes/pki/admin.crt | base64 -o w | base64 --decode # --> debe mostrar el valor integro del fichero

## Localizar el directorio oculto y mostrar el contenido:
cd /root/.kube ; ls -l
/root/.kube
total 12
drwxr-x--- 4 root root 4096 Jul 16 17:02 cache
-rw------- 1 root root 5636 Jul 16 17:02 config

## Localizar y filtrar un cluster dentro del fichero config:
cat config | grep cluster
clusters:
- cluster:
    cluster: kubernetes

## Localizar y filtrar un usuario dentro del fichero config:
cat config | grep user
    user: kubernetes-admin
users:
  user:

## Localizar y filtrar un contexto dentro del fichero config:
cat config | grep context
contexts:
- context:
current-context: kubernetes-admin@kubernetes

## Mostrar el fichero my-kube-config.yaml:
cat my-kube-config 
apiVersion: v1
kind: Config

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: development
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: kubernetes-on-aws
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

- name: test-cluster-1
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user

users:
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: test-user@development
preferences: {}

## Filtrar por cluster en el fichero my-kube-config:
cat my-kube-config | grep cluster
clusters:
  cluster:
  cluster:
  cluster:
- name: test-cluster-1
  cluster:
    cluster: development
    cluster: kubernetes-on-aws
    cluster: production
    cluster: test-cluster-1

## Filtrar por contexto:
cat my-kube-config | grep context
contexts:
  context:
  context:
  context:
  context:
current-context: test-user@development

## Localizar el cluster de módulo research 
 cat my-kube-config | grep -A2 research
- name: research
  context:
    cluster: test-cluster-1

## Localizar el cluster del módulo dev:
cat my-kube-config | grep  dev
- name: development
- name: test-user@development
    cluster: development
    user: dev-user
- name: dev-user
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
current-context: test-user@development

## Localizar el cluster del módulo aws:
cat my-kube-config | grep  aws
- name: kubernetes-on-aws
- name: aws-user@kubernetes-on-aws
    cluster: kubernetes-on-aws
    user: aws-user
- name: aws-user
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

## Localizar los contextos declarados en el fichero my-kube-config:
cat my-kube-config | grep -A2 -B2 context
    server: https://controlplane:6443

contexts:
- name: test-user@development
  context:
    cluster: development
    user: test-user

- name: aws-user@kubernetes-on-aws
  context:
    cluster: kubernetes-on-aws
    user: aws-user

- name: test-user@production
  context:
    cluster: production
    user: test-user

- name: research
  context:
    cluster: test-cluster-1
    user: dev-user
--
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key

current-context: test-user@development
preferences: {}

## Cambiar al contexto a research:
kubectl config --kubeconfig=/root/my-kube-config use-context research
Switched to context "research".

## Verificar el contexto en uso:
kubectl config --kubeconfig=/root/my-kube-config current-context 
research

## Editar el fichero bashrc, incluir a pie de página un export con la ruta al fichero config:
nano ~/.bashrc
export KUBECONFIG=/root/my-kube-config
source ~/.bashrc

## Verificar los pods, debido a que hay un bug en la ruta destacada, hay que solucionarlo:
kubectl get pods
error: unable to read client-cert /etc/kubernetes/pki/users/dev-user/developer-user.crt for dev-user due to open /etc/kubernetes/pki/users/dev-user/developer-user.crt: no such file or directory

## Localizar bug en la linea de certificados:
kubectl config view --kubeconfig=/root/my-kube-config | grep -A5 "name: dev-user"
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: test-user
  user:

## Editar el fichero, debemos llamarlo igual que el fichero situado en la ruta:
nano /root/my-kube-config
client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt --> X
client-certificate: /etc/kubernetes/pki/users/dev-user/dev-user.crt --> V
source ~/.bashrc

## Verificar si se ha corregido:
kubectl get pods
No resources found in default namespace.
