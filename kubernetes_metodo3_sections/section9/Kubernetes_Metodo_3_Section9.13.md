## 90.1º El objeto ingress tiene limitaciones a la hora de ofrecer sus funciones, en caso que dos empresas administran el objeto y tienen objetivos diferentes, necesitan una coordinación compleja que acabará en conflicto en el desarrollo a largo plazo. 

## 90.2º Las carencias del objeto ingress son: ["Muti-tenacidad","Aislamiento_del_NS","No_lleva_RBAC","Aislamiento_de_los_recursos"]

## 90.3º Otras limitaciones están relacionadas con el protocolo http; no admite el https. Depende de la coincidencia del host. Incluyendo: ["NO_puede_enrutar(TCP/UDP)","Control_de_tráfico","Autenticación","Limitación_de_Ratio","Redirigir_enlaces","Sobreescritura","Pronteción_contra_Middleware","Soporte_para_Puertos_Web","Personalizar_la_pantalla_de_error","Afinizado_de_la_Sesión","Compartir_el_origen_del_recurso"]

## 90.4º Para solventar estas situaciones, Kubernetes ha desarrollado el Gateway API, centrando en el enrutamiento. Ofrece soporte a: ["Ingress","Balancedador_de_carga","Servicio_para_API/MESH"]. Cuando dos empresas acceden a la misma estrcuctura de red, el gateway_api introduce tres objetos que son gestionados por adminsitradores distintos. El administrador de la infraestructura establece el gatewayClass, el operador de cluster establece el gateway y el desarrollador de aplicaciones el HTTPRoute, en este tipo de objeto se puede añadir más protocolos para la conexión. 

## 90.5º El objeto ingress se puede enlazar con un objeto gateway que lleve el tls compatible con el ingress, esta conexión solicita el puerto 443 con el https y httproute, incluyendo la redirección (la redirección se escribe en el yaml del ingress)

## 90.6º Otro tipo de despliegue del ingress es el despliegue carary-ingress, el 20% del tráfico es usado para la infraestructura principal, el 80% restante es para dirigirlo automáticamente a otros sercicios (Solo funciona con Nginx. Otros controladores no tienen soporte con estas directivas)

## 90.7º La Gateway Api ofrece una completa configuración visible desde un solo fichero, sin necesidad de configuraciones a parte, puede haber cambios entre las versiones v1/v2, permite dividir el tráfico de red en 80% la v1 y 20% la v2. No necesita anotaciones en el fichero. Permitiendo funcionar en cualquier despliegue de la Gateway API. 

## Docuemtación extra:
https://gateway-api.sigs.k8s.io/docs/introduction/
https://docs.nginx.com/nginx-gateway-fabric/install/helm/
https://gateway-api.sigs.k8s.io/guides/user-guides/http-routing/
https://gateway-api.sigs.k8s.io/guides/user-guides/http-redirect-rewrite/
https://gateway-api.sigs.k8s.io/guides/user-guides/http-header-modifier/
https://gateway-api.sigs.k8s.io/guides/user-guides/traffic-splitting/
https://gateway-api.sigs.k8s.io/guides/user-guides/http-request-mirroring/
https://gateway-api.sigs.k8s.io/guides/user-guides/tls/

## 90.8º En caso de no estar instalado el Gateway API, se inserta el comando de instalación
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.6.2" | kubectl apply -f -

helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway

## Instala el controlador de gateway y el recurso de personalización.

## 90.9º La clase gateway es implementada por un controlador. Objetivos: ["Despelgar_la_configuración_desde_la_implementación_en_curso"."Soporta_multiples_implementaciones_en_un_solo_cluster"]

## 90.10º El Gateway de Kubernetes es un recurso que define como el tráfico entra en el cluster. Los protocolos específicos, los puertos y las tablas de enrutamiento.
