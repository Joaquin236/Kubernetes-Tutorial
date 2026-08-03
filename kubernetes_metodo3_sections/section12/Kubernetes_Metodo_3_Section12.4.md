## 98.1º Las plantillas permiten a los ficheros de configuración ajustar los valores por separado. Permitiendo compartir los valores de las implementaciones
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
