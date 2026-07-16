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

##


##


##


