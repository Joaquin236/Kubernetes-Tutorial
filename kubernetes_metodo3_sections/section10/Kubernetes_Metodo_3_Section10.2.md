## 92.1º Cualquier sistema capacitado para virtualizar puede instalar y alojar servicios de Kubernetes. Normalmente este servicio lo ofrece los bancos de datos en la nube, accediendo desde el servidor web y administrarlo desde un web-dashboard.

## 92.2º Se puede instalar desde un Linux x64 Host y/o Virtual Machine. Para desplegar Kubernetes desde Windows. (No dispone de los ficheros binarios nativos para su instalación). Depende por completo de una aplicación de virtualización que ofrezca un Linux x64 virtualizado. Incluso los contenedores Docker se basan en sistemas Linux y debajo de los Pods hay sistemas opertaivos. 

## 92.3º Tabla comparativa con las opciones
|Opciones de formato Turnkey                        |Opciones de Servicios con alojamiento web  |
|---------------------------------------------------|-------------------------------------------|
|Debes preparar la MV                               |Kubernetes-como-Servicio                   |
|Debes configurar la MV                             |Las MV las mantiene el servidor            |
|Desdes usar los scripts para desplegar el Cluster  |El admin evalua la instalción              |
|Debes mantener las MV operativas                   |Las MV las gestiona el Operador de Sistemas|

## 92.4º Red Hat ofrece una herramienta para distribuir el sistema de Kubernetes. Open-Shift y dispone de una GUI para administrar el contenido. Vmware ofrece servicios cloud para desplegar el Kubernetes desde un servidor web. Vagrant ofrece scripts para administrar el sistema existente en cualquier proveedor de Kubernetes.

## 92.5º Google distribuye un servicio de alojamiento para trabajar en una infraestructura de Kubernetes, junto con el Google_Cloud. Amazon distribuye un alojamiento de Kubernetes con el Amazon Elastic.

## 92.6º Tabla con las valoraciones de las soluciones de virtualización
|Valoración de opciones del modo aprendizaje |
|--------------------------------------------|
| Oracle VirtualBOX|············-618["68.3%"]|
|VMWare WorkStation|·····-189["20.9%"]       |
|       Cloud_1-AWS|············-424["46.9%"]|
|       Cloud_2-GCP|·······-216["23.9%"]     |
|     Cloud_3-Azure|····-134["14.8%"]        |
|           Vagrant|···-125["13.8%"]         |
|           Hyper-V|·-7["0.8%"]              |
