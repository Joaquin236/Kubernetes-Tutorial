## Instalar el Gateway API
kubectl kustomize "https://github.com/nginx/nginx-gateway-fabric/config/crd/gateway-api/standard?ref=v1.5.1" | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created

## Desplegar el Nginx Gateway_1
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml
customresourcedefinition.apiextensions.k8s.io/clientsettingspolicies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/nginxgateways.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/nginxproxies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/observabilitypolicies.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/snippetsfilters.gateway.nginx.org created
customresourcedefinition.apiextensions.k8s.io/upstreamsettingspolicies.gateway.nginx.org created

## Desplegar el Nginx Gateway_2
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml
namespace/nginx-gateway created
serviceaccount/nginx-gateway created
clusterrole.rbac.authorization.k8s.io/nginx-gateway created
clusterrolebinding.rbac.authorization.k8s.io/nginx-gateway created
configmap/nginx-includes-bootstrap created
service/nginx-gateway created
deployment.apps/nginx-gateway created
gatewayclass.gateway.networking.k8s.io/nginx created
nginxgateway.gateway.nginx.org/nginx-gateway-config created

## Verificar el deploy realizado
kubectl get pods -n nginx-gateway
NAME                             READY   STATUS    RESTARTS   AGE
nginx-gateway-685ffdcf84-55bd4   2/2     Running   0          52s

## Consultar el yaml del servicio nginx-gateway
kubectl get svc -n nginx-gateway nginx-gateway -o yaml

## Actualizar el puerto del nginx-gateway con el comando aportado
kubectl patch svc nginx-gateway -n nginx-gateway --type='json' -p='[
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30081}
]'
service/nginx-gateway patched

## Crea un fichero de gateway para nginx-gateway
nano nginx-gateway_file.yaml 
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: nginx-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
     namespaces:
      from: All
kubectl apply -f nginx-gateway_file.yaml 
gateway.gateway.networking.k8s.io/nginx-gateway created

## El sistema ha desplegado un pod y un servicio, se necesita un httproute para seguir su configuración
kubectl get pods,svc -o wide 
NAME               READY   STATUS    RESTARTS   AGE   IP           NODE           NOMINATED NODE   READINESS GATES
pod/frontend-app   1/1     Running   0          10m   172.17.0.5   controlplane   [none]           [none]

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/frontend-svc   ClusterIP   172.20.47.182   [none]        80/TCP    10m   run=frontend-app
service/kubernetes     ClusterIP   172.20.0.1      [none]        443/TCP   41m   [none]

nano frontend-route.yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: frontend-route
  namespace: default
spec:
  parentRefs:
    - name: nginx-gateway           # Name of the Gateway
      namespace: nginx-gateway      # Namespace where the Gateway is deployed
      sectionName: http             # Attach to the 'http' listener
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: frontend-svc
          port: 80
kubectl apply -f frontend-route.yaml 
httproute.gateway.networking.k8s.io/frontend-route created

kubectl get httproute frontend-route 
kubectl describe httproute frontend-route
