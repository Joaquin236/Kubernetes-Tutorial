## 74.1º Kubectx es una herramienta descargable para administrar los comandos de kubernetes útil para cambiar los contextos de cada opción y argumento en los entornos multi-cluster. URL --> https://github.com/ahmetb/kubectx

## 74.2º Para instalarlo usaremos dos comandos:
git clone https//github.com/ahmetb/kubectx /opt/kubectx
ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
kubectx # --> listar los contextos del comando
kubectx ["context_name"] # --> cambiar a un contexto nuevo
kubectx - # --> regresar a otro contexto
kubectx -c # --> mostrar el contexto actual

## 74.3º Otra herramienta es Kubens, esta herramienta permite cambiar a los usuarios entre los espacios de nombres con un solo comando

## 74.4º Para instalarlo usaremos dos comandos:
git clone https://github.com/ahmetb/kubectx /opt/kubectx
ln -s /opt/kubectx/kubens /usr/local/bin/kubens
kubens ["new_namespace"] # --> cambiar a un nuevo ns
kubens - # retroceder al namespace anterior
