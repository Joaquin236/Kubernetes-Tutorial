## 78.1º Las primeras versiones de Kubernetes usaban el Container-D como motor de los contenedores, el sistema estaba dentro del nucleo de Kubernetes, cuando llegaron el motor Cri-O y RKT necesitaron abrir la compatibilidad y el motor fue reubicado fuera del código fuente.

## 78.2º La interfaz de rutina de contenedore es un estándar actual que define la orquestación de los contenedores, si se desarrollase más adelante alguna interfaz nueva, debe ser más fácil adaptarla siguiendo las normas del componente RKT sin modificar el código fuente de la aplicación principal.

## 78.3º La infraestructura de Kubernetes se amplía con los servicios de nube de proveedores de servicios de red.

## 78.4º La unidades de almacenamiento se pueden subir a la nube a través de plugings de proveedores de servicios cloud, el servicio cloud debe conectarse a una cuenta para acceder al banco de datos válido y cuando se desconecte debe guardarse todo en la ruta donde se configuró el contenedor.

## 78.5º La interfaz de almacenamiento de contenedores tiene como objetivo adaptarse a cualquier herramienta de orquestar contenedores. Si el orquestador tiene que desplegar un pod y no tiene el volumen disponible. Lo tiene que crear y activar