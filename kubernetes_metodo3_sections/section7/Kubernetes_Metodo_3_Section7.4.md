## 64.1º Kubernetes agrupa las api en función de las tareas que realiza, pueden ser: ["/metrics","/healthz","/version","/api","/apis","/logs"], La url completa es: https://172.17.0.42:6443/["name"].

## 64.2º ["/metrics","/healthz"] --> son los monitores de integridad del cluster

## 64.3º ["/version"] --> muestra la versión del componente instalado

# 64.4º ["/logs"] --> Muestra y recopila los registros de cada aplicación

## 64.5º ["/api"] Es el nucleo de las funciones de kubernetes, dentro hay un ramal donde se ubican: ["namespaces","pods","rc"],["events","endpoints","nodes"],["bindings","PV","PVC"],["configmaps","secrets","services"].

## 64.6º ["/apis"] Contiene los complementos del sistema: ["/apps","/extensions","/networking.k8s.io","/storage.k8s.io","/authentication.k8.io","/certification.k8s.io"]. Dentro de Apps se ubica: ["/deployments","/replicasets","/statefulsets"] --> Resources. Dentro de deployments, localizamos los verbs, los valores de los comandos de interacción: ["list","get","create","delete","update","watch"]. Dentro de networking.k8s.io está el sistema de networking y dentro de certificates.k8s.io está el certificate-signing-requests.

## 64.7º Si usamos curl https://localhost:6443 -k se mostrará los paths de la estructura. Si filtramos con grep "name" se mostrará agrupado por nombre: curl https://localhost:6443/apis -k | grep "name".

## 64.8º Si autenticamos el usuario a través del comando: curl https://localhost:6443 -k --key admin.key --cert admin.crt --cacert ca.crt mostramos los paths de las funciones de kubernetes.

## 64.9º El kube-proxy es un servicio que incluye el servicio http para las conexiones entre los pods y el cluster, mientras el kucectl proxy es un comando para usar el proxy local con el puerto 8001 y accede al cluser.

