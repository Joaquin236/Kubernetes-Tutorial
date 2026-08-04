## 98.1º Los repositorios que se acceden con el comando administran lo que se inserta con el comando helm. Al aplicar el contenido del repositorio al cluster, se crea una versión de la aplicación. Dentro de cada uno lleva las reviciones de versiones para alternar entre cada una de ellas. Cada imagen se almacena en los contenedore Docker, permitiendo desplegarlas al cluster de Kubernetes.

## 98.2º  Las plantillas permiten a los ficheros de configuración ajustar los valores por separado. Permitiendo compartir los valores de las implementaciones. Parte del objetivo es evitar modificar el fichero.yaml de los despliegues, solo hay que editar la plantilla con el valor del ["replicaCount" y del "image.repository"]
nano service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-world
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app: hello-world
kubectl apply -f service.yaml
---
nano values.yaml
replicanCount: 1
image:
  repository: nginx
---
nano deploy_multiuser.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  replicas: "{{ .Values.replicaCount}}"
  selector:
    matchLabels:
      app: hello-world
  template:
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository}}"
          ports:
            - name: http
              containerPort: 80
              protocol: TCP 
kubectl apply -f deploy_multiuser.yaml

## 98.3º Incluso es posible llevar todos los valores del despliegue.yaml a ficheros con los valores de cada variable para no tener que editar el fichero prinicipal, pero puede ser más difícil administrarlo. Para aplicar este fichero se necesita una cierta cantidad de ficheros que contiene los valores pre-establecidos, cada fichero tiene un valor que se administra de forma modular
nano super_deploy.yaml/deployment.yaml
apiVersion: {{ include "common.capacibilites.deployment.apiVerion" . }}
kind: Deployment
metadata:
  name: {{ include "common.names.fullname" . }}
  namespace: {{ .Release.Namespace | quote }}
  labels: {{- include "common.labels.standard" . | nindent 4 }}
    {{- if .Values.commonLables }}
    {{- include "common.tplvalues.render" ( dict "value" .Values.commonLabels "context" $)  }}
    {{- end}}
    {{- if .Values.commonAnnotations }}
  annotations: {{- include "common.tpvalues.render" (dict "value" .Values.commonAnnotations) }}
spec:
  selector:
    matchLabels: {{- include "common.labels.matchLabels" . | nindent 6 }}
  {{- if .Values.updateStrategy }}
  strategy: {{- toYaml .Values.updateStrategy | nindent 4 }}
  {{- end }}
  {{- end }}
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount}}
kubectl apply -f super_deploy.yaml/deployment.yaml
## Los ficheros necesarios son: ["config-secret.yaml","deployment.yaml/super_deploy.yaml","externaldb-secrets.yaml","extra-list.yaml","hpa.yaml","httpd-configmap.yaml","ingress.yaml","metrics-svc.yaml","pdb.yaml","postinit.yaml","pvc.yaml","secrets.yaml","servicemonitor.yaml","svc.yaml","tls-secrets.yaml"]

## 98.4º Otros comandos para administrar un servicio desde Helm
helm install my-site bitnami/wordpress
helm install bitnami/wordpress
helm install my-SECOND-site bitnami/wordpress

## 98.5º Si queremos desplegar otros servicios también se pueden realizar con Helm, hay todo tipo de servicios alojados en repositorios para desplegar con helm, todos se almacenan en Helm.ArtifactHub.io para ser descargados por la herramienta. Web oficial de Helm_Artifact_Hub --> https://artifacthub.io/

## 98.6º Algunas sugerencias disponibles del Artifact_Hub son: ["appscode","community_operators","Truecharts","Bitnami"]

## 99.1º La esctructura de los charts es
hello-world-chart 
├── templates.dir # --> Templates directory
├── values.yaml   # --> Configurable values
├── Chart.yaml    # --> Chart Information
├── LICENSE.txt   # --> Chart License
├── README.md     # --> Readme File
└── charts.dir    # --> Dependency Charts
