## 104.1º La herramienta alternativa a Kustomize es Helm. 
- Helm crea las plantillas para permitir asignar variables a las propiedades.
- Helm es mucho más que personalizar los parámetros, modifica el entorno básico. Helm también es un adminitrador de paquetes para la aplicación.
- Helm ofrece características extra como conficionales, bucles, funciones y conectores.
- Helm no verifica la estructura sintáctica del fichero yaml. Es difícil leer los ficheros cuando se hacen más grandes

## 104.2º Cuando usamos esta herramienta los ficheros se configuran de esta forma
nano deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Values.name }}
  template:
    metadata: 
      labels:
        app: {{ .Values.name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "nginx:{{ .Values.image.tag }}"
---
nano values.yaml
replicaCount: 1
image:
  tag: "2.4.4"
helm install -f deploy.yaml
