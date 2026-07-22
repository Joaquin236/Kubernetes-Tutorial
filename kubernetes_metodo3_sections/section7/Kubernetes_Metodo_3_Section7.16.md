## 76.1º Crear un fichero yaml con la apiversion diferente a la habitual
nano flight_ticket.yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Munbai
  to: London
  number: 2

## 76.2º Crear un fichero yaml que de soporte de acceso al billete de vuelo
nano flight_ticket_custom_definition.yaml
apiVerion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced
  group: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
      - ft
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
  plural: flightticket
  shortNames:
    - ft
versions:
  - name: v1
  served: true
  storage: true
  schema:
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          properties:
            from:
              type: string
            to:
              type: string
            number:
              type: integer
              minimum: 1
              maximum: 10

## 76.3º Ordena la creación declarativa de los componentes creados en los ficheros
kubectl create -f flight_ticket_custom_definition.yaml ; kubectl create -f flight_ticket.yaml

## 76.4º Consultar recusros de la api
kubectl api-resources 
kubectl get ft
kubectl get flightticket

## 76.5º Borrar los recursos personalizados
kubectl delete flight_ticket_custom_definition.yaml ; kubectl delete flight_ticket.yaml

##
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.kodekloud.com
spec:
  group: datasets.kodekloud.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
    - int
kubectl create -f crd.yaml 
customresourcedefinition.apiextensions.k8s.io/internals.datasets.kodekloud.com created

##
nano custom.yaml
kind: Internal
apiVersion: datasets.kodekloud.com/v1
metadata:
  name: internal-space
  namespace: default
spec:
  internalLoad: "high"
  range: 80
  percentage: "50"
kubectl create -f custom.yaml 
internal.datasets.kodekloud.com/internal-space created

##
kubectl describe crd collectors.monitoring.controller | grep -A5 Image
              Image:
                Type:  string
              Name:
                Type:  string
              Replicas:
                Type:  integer
##
nano datacenter.yaml
kind: Global
apiVersion: traffic.controller/v1
metadata:
  name: datacenter
  namespace: default
spec:
  dataField: 2
  access: true
kubectl create -f datacenter.yaml 
global.traffic.controller/datacenter created

##
kubectl describe crd internals.datasets.kodekloud.com
kubectl describe crd globals.traffic.controller
kubectl describe crd collectors.monitoring.controller

##
kubectl describe crd globals.traffic.controller | grep -A1 Short
    Short Names:
      gb
--
    Short Names:
      gb
      