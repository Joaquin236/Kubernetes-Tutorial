## 90.1º El objeto ingress tiene limitaciones a la hora de ofrecer sus funciones, en caso que dos empresas administran el objeto y tienen objetivos diferentes, necesitan una coordinación compleja que acabará en conflicto en el desarrollo a largo plazo. 

## 90.2º Las carencias del objeto ingress son: ["Muti-tenacidad","Aislamiento_del_NS","No_lleva_RBAC","Aislamiento_de_los_recursos"]

## 90.3º Otras limitaciones están relacionadas con el protocolo http; no admite el https. Depende de la coincidencia del host. Incluyendo: ["NO_puede_enrutar(TCP/UDP)","Control_de_tráfico","Autenticación","Limitación_de_Ratio","Redirigir_enlaces","Sobreescritura","Pronteción_contra_Middleware","Soporte_para_Puertos_Web","Personalizar_la_pantalla_de_error","Afinizado_de_la_Sesión","Compartir_el_origen_del_recurso"]

## 90.4º Para solventar estas situaciones, Kubernetes ha desarrollado el Gateway API, centrando en el enrutamiento. Ofrece soporte a: ["Ingress","Balancedador_de_carga","Servicio_para_API/MESH"]. Cuando dos empresas acceden a la misma estrcuctura de red, el gateway_api introduce tres objetos que son gestionados por adminsitradores distintos. El administrador de la infraestructura establece el gatewayClass, el operador de cluster establece el gateway y el desarrollador de aplicaciones el HTTPRoute, en este tipo de objeto se puede añadir más protocolos para la conexión. 

## 90.5º El objeto ingress se puede enlazar con un objeto gateway que lleve el tls compatible con el ingress, esta conexión solicita el puerto 443 con el https y httproute, incluyendo la redirección (la redirección se escribe en el yaml del ingress)

## 90.6º Otro tipo de despliegue del ingress es el despliegue carary-ingress, el 20% del tráfico es usado para la infraestructura principal, el 80% restante es para dirigirlo automáticamente a otros sercicios (Solo funciona con Nginx. Otros controladores no tienen soporte con estas directivas)

## 90.7º La Gateway Api ofrece una completa configuración visible desde un solo fichero, sin necesidad de configuraciones a parte, puede haber cambios entre las versiones v1/v2, permite dividir el tráfico de red en 80% la v1 y 20% la v2. No necesita anotaciones en el fichero. Permitiendo funcionar en cualquier despliegue de la Gateway API. 

## Docuemtación extra:
- https://gateway-api.sigs.k8s.io/docs/introduction/
- https://docs.nginx.com/nginx-gateway-fabric/install/helm/
- https://gateway-api.sigs.k8s.io/guides/user-guides/http-routing/
- https://gateway-api.sigs.k8s.io/guides/user-guides/http-redirect-rewrite/
- https://gateway-api.sigs.k8s.io/guides/user-guides/http-header-modifier/
- https://gateway-api.sigs.k8s.io/guides/user-guides/traffic-splitting/
- https://gateway-api.sigs.k8s.io/guides/user-guides/http-request-mirroring/
- https://gateway-api.sigs.k8s.io/guides/user-guides/tls/

## 90.8º En caso de no estar instalado el Gateway API, se inserta el comando de instalación
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.6.2" | kubectl apply -f -

helm install ngf oci://ghcr.io/nginx/charts/nginx-gateway-fabric --create-namespace -n nginx-gateway

## Instala el controlador de gateway y el recurso de personalización.

## 90.9º La clase gateway es implementada por un controlador. Objetivos: ["Despelgar_la_configuración_desde_la_implementación_en_curso"."Soporta_multiples_implementaciones_en_un_solo_cluster"]

## 90.10º El Gateway de Kubernetes es un recurso que define como el tráfico entra en el cluster. Los protocolos específicos, los puertos y las tablas de enrutamiento.

## 90.11º El HTTPRoute define como el tráfico es permitido en los servicios de Kubernetes. Funciona en conjunto con el Gateway API para las peticiones basadas en reglas.

## 90.12º Las redirecciones y sobreescrituras son herramientas poderosas para modificar las respuestas/peticiones inminentes, antes de llegar al servicio backend.

## 90.13º Las cabeceras HTTP se pueden modificar según las respuestas: ["add","set","remove"]

## 90.13º El filtro de tráfico permite distribuirlo entre multiles servicios de backend. Esto es ofrecido en canary-deployments.

## 90.14º La respuesta en espejo permite enviar una copia del servicio inminente a otro servicio para verificar/evaluar sin afectar al servicio primario.

## 90.15º La capa de seguridad [TLS] es usada para encriptar tráfico entre clientes y servidores, asegurando la conexión frente a los ataques. A través del Gateway se puede establecer esta capa usando un certificado alojado en el kubernetes-secret. Los servicios backend reciben el contenido desencriptado para que le resulte legible.

## 90.16º El Gateway API soporta mucho más que el tráfico HTTP. Puedes configurarlo con los protocolos: ["TCP","UDP","gRPC"]. La flexibilidad que ofrece da soporte a diversas aplicaciones, bases de datos y microservicios.

## 90.17º Muestra de un fichero Ingress estándar
nano Ingress-wear-watch.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  annotations: wear.my-online-store.com
 nginx.ingress.kubernetes.io/ssl-redirect: "true"
 nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
rules:
- host: wear.my-online-store.com
  http:
    path: /foo
    backend:
      serviceName: wear-service
      servicePort: 80
- host: watch.my-online-store.com
  http:
    paths:
    - backend:
        serviceName: watch-service
        servicePort: 80
kubectl apply -f Ingress-wear-watch.yaml

## 90.18º Muestra de fichero ingress-cors para Nginx
nano Ingress-cors.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cors-ingress
  annotations:
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, PUT, POST"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://allowed-origin.com"
    nginx.ingress.kubernetes.io/cors-allow-credentials: "true"
kubectl apply -f Ingress-cors.yaml

## 90.19º Muestra de fichero ingress estándar para Traefik
nano Ingress_stand_Traefik.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: traefik-ingress
  annotations:
    traefik.ingress.kubernetes.io/headers.customresponseheaders:
     Acces-Control-Allow-Origin: '*'
     Acces-Control-Allow-Methods: GET,POST,PUT,DELETE,OPTIONS
     Acces-Control-Allow-Headers: Content-Type,Autorization
     Acces-Control-Allow-Credentials: "true"
     Acces-Control-Max-Age: 3600
kubectl apply -f Ingress_stand_Traefik.yaml

## 90.20º Muestra de fichero http-route.yaml
nano http-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
  - name: example-gateway
  hostsnames:
  - "www.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /login
    backendRefs:
    - name: example-svc
      port: 8080
kubectl apply -f http-route.yaml

## 90.21º Muesta del fichero gateway-class.yaml
nano gateway-class.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: example-class
spec:
  controllerName: example.com/gateway-controller
kubectl apply -f gateway-class.yaml

## 90.22º Muestra del fichero gateway.yaml
nano gateway.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: example-gateway
  listernes:
  - name: http
    protocol: HTTP
    port: 80
kubectl apply -f gateway.yaml



