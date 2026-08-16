## Consultar el ns mc*
kubectl get namespaces | grep mc
mc-namespace      Active   53s

## Cambiar de ns
kubectl config set-context --current --namespace=mc-namespace 
Context "kubernetes-admin@kubernetes" modified.

## Crea un pod-desplegable
nano mc-pod-deploy.yaml
apiVersion: v1
kind: Pod
metadata:
  name: mc-pod
  namespace: mc-namespace
spec:
  containers:
    - name: mc-pod-1
      image: nginx:1-alpine
      env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
    - name: mc-pod-2
      image: busybox:1
      volumeMounts:
        - name: shared-volume
          mountPath: /var/log/shared
      command:
        - "sh"
        - "-c"
        - "while true; do date >> /var/log/shared/date.log; sleep 1; done"
    - name: mc-pod-3
      image: busybox:1
      command:
        - "sh"
        - "-c"
        - "tail -f /var/log/shared/date.log"
      volumeMounts:
        - name: shared-volume
          mountPath: /var/log/shared
  volumes:
    - name: shared-volume
      emptyDir: {}
kubectl apply -f mc-pod-deploy.yaml 
pod/mc-pod created

## Entra con el node01 y realiza una instalación
username: bob
password: caleston123
ssh bob@node01
insert pass:
sudo su / sudo -i
dpkg -i /root/cri-docker_0.3.16.3-0.debian.deb ; systemctl start cri-docker ; systemctl enable cri-docker ; systemctl is-active cri-docker ; systemctl is-enabled cri-docker
Selecting previously unselected package cri-dockerd.
(Reading database ... 18376 files and directories currently installed.)
Preparing to unpack .../cri-docker_0.3.16.3-0.debian.deb ...
Unpacking cri-dockerd (0.3.16~3-0~ubuntu-jammy) ...
Setting up cri-dockerd (0.3.16~3-0~ubuntu-jammy) ...
Created symlink /etc/systemd/system/multi-user.target.wants/cri-docker.service → /lib/systemd/system/cri-docker.service.
Created symlink /etc/systemd/system/sockets.target.wants/cri-docker.socket → /lib/systemd/system/cri-docker.socket.
/usr/sbin/policy-rc.d returned 101, not running 'start cri-docker.service cri-docker.socket'
active
enabled

## Localiza todos los CRD del sistema y guarda el resultado en /root/vpa-crds.txt
kubectl get crd -o custom-columns=NAME:.metadata.name | grep verticalpodautoscaler > /root/vpa-crds.txt
cat vpa-crds.txt 
verticalpodautoscalercheckpoints.autoscaling.k8s.io
verticalpodautoscalers.autoscaling.k8s.io

## Regresa al ns default
kubectl config set-context --current --namespace=default 
Context "kubernetes-admin@kubernetes" modified.

## El pod messaging debe estar expuesto a este puerto
kubectl expose pod messaging --port=6379 --name messaging-service

## Crea un servicio para messaging
nano messaging-service.yaml
apiVersion: v1
kind: Service
metadata: 
  name: messaging-service
  labels:
    app: messaging
spec:
  ports:
    - port: 6379
  selector:
    app: messaging
  type: ClusterIP
kubectl apply -f messaging-service.yaml 
service/messaging-service created

## Crea un despliegue con los valores establecidos
nano /root/hr-web-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hr-web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hr-web-app
  template:
    metadata: 
      labels:
        app: hr-web-app
    spec:
      containers:
        - name:  hr-web-app
          image: kodekloud/webapp-color
kubectl apply -f /root/hr-web-app.yaml
deployment.apps/hr-web-app created

## Describe el pod orange. Verifica esta sección del pod, está incorrecto
kubectl describe pods orange | grep -A4 Command
    Command:
      sh
      -c
      sleeeep 2;
    State:          Terminated

## Exporta el pod, editalo, borra el anterior y vuelve a aplicarlo
kubectl get pods orange -o yaml > /root/orange_pod.yaml
nano /root/orange_pod.yaml
kubectl delete pods orange 
pod "orange" deleted from default namespace
kubectl apply -f /root/orange_pod.yaml 
pod/orange created
kubectl get pods orange -o wide 
NAME     READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
orange   1/1     Running   0          25s   172.17.0.13   controlplane   <none>           <none>

## El deploy debe estar expuesto al puerto 8008
kubectl expose deployment hr-web-app --type=NodePort --port=8080 --name=hr-web-app-service --dry-run=client -o yaml > hr-web-app-service.yaml 

## Crea un servicio para hr-web
nano hr-web-app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hr-web-app-service
spec:
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: hr-web-app
  type: NodePort
status:
  loadBalancer: {}
kubectl apply -f hr-web-service.yaml
service/hr-web-app created

## Crea un persistent vol para almacenamiento de recursos
nano pv_analytics.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-analytics
spec:
  capacity:
    storage: 100Mi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  hostPath:
      path: /pv/data-analytics
kubectl apply -f pv_analytics.yaml
persistentvolume/pv-analytics created

## Crea un escalador horizontal
nano horizontal_scale_hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: webapp-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: kkapp-deploy
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
kubectl apply -f horizontal_scale_hpa.yaml 
horizontalpodautoscaler.autoscaling/webapp-hpa created

## Crea un escalador vertical 
kubectl create -n default -f - <<EOF
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: analytics-vpa
  namespace: default
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: analytics-deployment
  updatePolicy:
    updateMode: "Recreate"
EOF
verticalpodautoscaler.autoscaling.k8s.io/analytics-vpa created

## Crea el gateway para nginx
kubectl create -n nginx-gateway -f - <<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      protocol: HTTP
      port: 80
EOF
gateway.gateway.networking.k8s.io/web-gateway created

## Un co-administrador desplegó la carta para help kk-mock1 en el ns kk-ns en el cluster. Necesitamos actualizar la carta del helm y nuestro equipo quiere que actualices el repositorio helm para recibir cambios
helm ls -A ; helm repo ls ; helm repo update kk-mock1 -n kk-ns ; helm search repo kk-mock1/podinfo -n kk-ns -l | head -n30 ; helm upgrade kk-mock1 kk-mock1/podinfo -n kk-ns --version=6.11.2 ; helm ls -n kk-ns
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
kk-mock1        kk-ns           2               2026-08-15 15:14:21.145405915 +0000 UTC deployed        podinfo-6.11.2  6.11.2     
NAME            URL                                   
kk-mock1        https://stefanprodan.github.io/podinfo
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "kk-mock1" chart repository
Update Complete. ⎈Happy Helming!⎈
NAME                    CHART VERSION   APP VERSION     DESCRIPTION                      
kk-mock1/podinfo        6.14.1          6.14.1          Podinfo Helm chart for Kubernetes
kk-mock1/podinfo        6.14.0          6.14.0          Podinfo Helm chart for Kubernetes
kk-mock1/podinfo        6.13.0          6.13.0          Podinfo Helm chart for Kubernetes
Release "kk-mock1" has been upgraded. Happy Helming!
NAME: kk-mock1
LAST DEPLOYED: Sat Aug 15 15:21:13 2026
NAMESPACE: kk-ns
STATUS: deployed
REVISION: 3
NOTES:
1. Get the application URL by running these commands:
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl -n kk-ns port-forward deploy/kk-mock1-podinfo 8080:9898
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
kk-mock1        kk-ns           3               2026-08-15 15:21:13.198193566 +0000 UTC deployed        podinfo-6.11.2  6.11.2     
