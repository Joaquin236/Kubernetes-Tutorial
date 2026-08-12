## Localizar algún mensaje de error del kubelet.service
sudo journalctl -u kubelet.service 
Aug 12 12:39:10 controlplane kubelet[1921]: E0812 12:39:10.937320 # --> failed to load Kubelet config file /var/lib/kubelet/config.yaml

## Consultar los pods en el ns kube-system
kubectl get pods -n kube-system -o wide 
NAME                                   READY   STATUS             RESTARTS      AGE     IP              NODE           NOMINATED NODE   READINESS GATES
coredns-6f6c7df987-9pxj8               1/1     Running            0             13m     172.17.0.3      controlplane   [none]           [none]
coredns-6f6c7df987-qwmmt               1/1     Running            0             13m     172.17.0.2      controlplane   [none]           [none]
etcd-controlplane                      1/1     Running            0             13m     10.244.170.83   controlplane   [none]           [none]
kube-apiserver-controlplane            1/1     Running            0             13m     10.244.170.83   controlplane   [none]           [none]
kube-controller-manager-controlplane   1/1     Running            0             13m     10.244.170.83   controlplane   [none]           [none]
kube-proxy-mb4tn                       1/1     Running            0             13m     10.244.170.83   controlplane   [none]           [none]
kube-scheduler-controlplane            0/1     CrashLoopBackOff   6 (42s ago)   6m26s   10.244.170.83   controlplane   [none]           [none]

## Consultar los componentes de todos los espacios de nombres
kubectl get componentstatuses -o wide -A
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                        ERROR
scheduler            Unhealthy   Get "https://127.0.0.1:10259/healthz": dial tcp 127.0.0.1:10259: connect: connection refused   
controller-manager   Healthy     ok                                                                                             
etcd-0               Healthy     ok     

## Consultar algún registro del kube-scheduler
kubectl logs -n kube-system kube-scheduler-controlplane # --> There aren't logs at the pod

## Describir los eventos sucedidos en el kube-scheduler
kubectl describe pods -n kube-system kube-scheduler-controlplane | grep -A10 Event
Events:
  Type     Reason   Age                    From     Message
  ----     ------   ----                   ----     -------
  Normal   Pulled   2m14s (x6 over 5m10s)  kubelet  spec.containers{kube-scheduler}: Container image "registry.k8s.io/kube-scheduler:v1.35.0" already present on machine and can be accessed by the pod
  Normal   Created  2m14s (x6 over 5m10s)  kubelet  spec.containers{kube-scheduler}: Container created
  Warning  Failed   2m14s (x6 over 5m9s)   kubelet  spec.containers{kube-scheduler}: Error: failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
  Warning  BackOff  63s (x18 over 5m8s)    kubelet  spec.containers{kube-scheduler}: Back-off restarting failed container kube-scheduler in pod kube-scheduler-controlplane_kube-system(186c52b5584d78a99894b5f06124bf5d)

## Localizar en el yaml del kube-scheduler
kubectl get pods -n kube-system kube-scheduler -o yaml | grep -A1 -B2 command
spec:
  containers:
  - command:
    - kube-schedulerrrr

## Verificar el replicaset activo
kubectl get replicasets.apps -o wide 
NAME           DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
app-5497957c   1         1         0       26s   nginx        nginx:alpine   app=app,pod-template-hash=5497957c

## Describir el replicaset localizado
kubectl describe replicaset app-5497957c
Name:           app-5497957c
Namespace:      default
Selector:       app=app,pod-template-hash=5497957c
Labels:         app=app
                pod-template-hash=5497957c
Annotations:    deployment.kubernetes.io/desired-replicas: 1
                deployment.kubernetes.io/max-replicas: 2
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/app
Replicas:       1 current / 1 desired
Pods Status:    0 Running / 1 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=app
           pod-template-hash=5497957c
  Containers:
   nginx:
    Image:         nginx:alpine
    Port:          [none]
    Host Port:     [none]
    Environment:   [none]
    Mounts:        [none]
  Volumes:         [none]
  Node-Selectors:  [none]
  Tolerations:     [none]
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  64s   replicaset-controller  Created pod: app-5497957c-w4r4r

## No es necesario extraer el yaml del pod defectuoso. Al ser un pod estático el yaml está en /etc/kubernetes/manifests/kube-scheduler.yaml y corregirlo
nano /etc/kubernetes/manifests/kube-scheduler.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    image: registry.k8s.io/kube-scheduler:v1.35.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /livez
        port: probe-port
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-scheduler
    ports:
    - containerPort: 10259
      name: probe-port
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        host: 127.0.0.1
        path: /readyz
        port: probe-port
        scheme: HTTPS
      periodSeconds: 1
      timeoutSeconds: 15
    resources:
      requests:
        cpu: 100m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /livez
        port: probe-port
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status: {}

## La corrección es automática, volvemos a consultar el pod en el ns kube-system
kubectl get pods -n kube-system -o wide 
NAME                                   READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
coredns-6f6c7df987-kn8fp               1/1     Running   0          20m   172.17.0.3       controlplane   [none]           [none]
coredns-6f6c7df987-rfrsl               1/1     Running   0          20m   172.17.0.2       controlplane   [none]           [none]
etcd-controlplane                      1/1     Running   0          20m   10.244.105.173   controlplane   [none]           [none]
kube-apiserver-controlplane            1/1     Running   0          20m   10.244.105.173   controlplane   [none]           [none]
kube-controller-manager-controlplane   1/1     Running   0          20m   10.244.105.173   controlplane   [none]           [none]
kube-proxy-bbms2                       1/1     Running   0          20m   10.244.105.173   controlplane   [none]           [none]
kube-scheduler-controlplane            1/1     Running   0          67s   10.244.105.173   controlplane   [none]           [none]

## Consulta el deploy en todos los espacios de nombres
kubectl get deploy -A -o wide 
NAMESPACE     NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
default       app       1/1     1            1           16m   nginx        nginx:alpine                              app=app
kube-system   coredns   2/2     2            2           23m   coredns      registry.k8s.io/coredns/coredns:v1.10.1   k8s-app=kube-dns

## El Deploy app solo tiene 1 replica, necesitamos dos para completarlo
kubectl get deployments.apps app -o yaml > app_file.yaml

##
nano app_file.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2026-08-12T14:50:04Z"
  generation: 1
  labels:
    app: app
  name: app
  namespace: default
  resourceVersion: "1678"
  uid: 76522f52-6234-4563-98a4-4528e506596a
spec:
  progressDeadlineSeconds: 600
  replicas: 2
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: app
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - image: nginx:alpine
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  availableReplicas: 1
  conditions:
  - lastTransitionTime: "2026-08-12T15:02:56Z"
    lastUpdateTime: "2026-08-12T15:02:56Z"
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: "2026-08-12T15:02:56Z"
    lastUpdateTime: "2026-08-12T15:02:56Z"
    message: ReplicaSet "app-5497957c" has successfully progressed.
    reason: NewReplicaSetAvailable
    status: "True"
    type: Progressing
  observedGeneration: 1
  readyReplicas: 1
  replicas: 2
  terminatingReplicas: 0
  updatedReplicas: 1

## Aplica el cambio para tener dos replicas operativas
kubectl apply -f app_file.yaml
Warning: resource deployments/app is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/app configured

## El despliegue solo tiene una replica activa
kubectl get deploy -A -o wide 
NAMESPACE     NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
default       app       1/2     1            1           19m   nginx        nginx:alpine                              app=app
kube-system   coredns   2/2     2            2           26m   coredns      registry.k8s.io/coredns/coredns:v1.10.1   k8s-app=kube-dns

## Consulta los pods activos en el ns kube-system
kubectl get pods -n kube-system -o wide 
NAME                                   READY   STATUS             RESTARTS       AGE     IP               NODE           NOMINATED NODE   READINESS GATES
coredns-6f6c7df987-kn8fp               1/1     Running            0              28m     172.17.0.3       controlplane   [none]           [none]
coredns-6f6c7df987-rfrsl               1/1     Running            0              28m     172.17.0.2       controlplane   [none]           [none]
etcd-controlplane                      1/1     Running            0              28m     10.244.105.173   controlplane   [none]           [none]
kube-apiserver-controlplane            1/1     Running            0              28m     10.244.105.173   controlplane   [none]           [none]
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   5 (112s ago)   4m51s   10.244.105.173   controlplane   [none]           [none]
kube-proxy-bbms2                       1/1     Running            0              28m     10.244.105.173   controlplane   [none]           [none]
kube-scheduler-controlplane            1/1     Running            0              8m25s   10.244.105.173   controlplane   [none]           [none]

## El pod está buscando un fichero que no existe en el sistema, el path debe ser corregido para que apunte al path correcto
kubectl logs -n kube-system kube-controller-manager-controlplane
I0812 15:09:27.021088       1 serving.go:386] Generated self-signed cert in-memory
E0812 15:09:27.021169       1 run.go:72] "command failed" err="stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory"

##
nano /etc/kubernetes/manifests/kube-controller-manager.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=172.17.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=172.20.0.0/16
    - --use-service-account-credentials=true
    image: registry.k8s.io/kube-controller-manager:v1.35.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: probe-port
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-controller-manager
    ports:
    - containerPort: 10257
      name: probe-port
      protocol: TCP
    resources:
      requests:
        cpu: 200m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: probe-port
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      name: flexvolume-dir
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /etc/kubernetes/controller-manager.conf
      name: kubeconfig
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      type: DirectoryOrCreate
    name: flexvolume-dir
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /etc/kubernetes/controller-manager.conf
      type: FileOrCreate
    name: kubeconfig
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}

## Tras corregirlo el pod ya puede administrar las replicas
kubectl get pods -n kube-system -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
coredns-6f6c7df987-kn8fp               1/1     Running   0          31m   172.17.0.3       controlplane   [none]           [none]
coredns-6f6c7df987-rfrsl               1/1     Running   0          31m   172.17.0.2       controlplane   [none]           [none]
etcd-controlplane                      1/1     Running   0          31m   10.244.105.173   controlplane   [none]           [none]
kube-apiserver-controlplane            1/1     Running   0          31m   10.244.105.173   controlplane   [none]           [none]
kube-controller-manager-controlplane   1/1     Running   0          53s   10.244.105.173   controlplane   [none]           [none]
kube-proxy-bbms2                       1/1     Running   0          31m   10.244.105.173   controlplane   [none]           [none]
kube-scheduler-controlplane            1/1     Running   0          11m   10.244.105.173   controlplane   [none]           [none]

## El siguiente error hace referencia que el pod está buscando el certificado ca.crt y no lo encontró
kubectl logs -n kube-system kube-controller-manager-controlplane
I0812 15:15:51.980184       1 serving.go:386] Generated self-signed cert in-memory
E0812 15:15:51.981642       1 run.go:72] "command failed" err="unable to load client CA provider: open /etc/kubernetes/pki/ca.crt: no such file or directory"

## Edita el fichero para corregir la ruta mal calibrada. El cambio lo recibe automáticamente después editar el fichero
nano /etc/kubernetes/manifests/kube-controller-manager.yaml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: kube-controller-manager
    tier: control-plane
  name: kube-controller-manager
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=172.17.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=172.20.0.0/16
    - --use-service-account-credentials=true
    image: registry.k8s.io/kube-controller-manager:v1.35.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: probe-port
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    name: kube-controller-manager
    ports:
    - containerPort: 10257
      name: probe-port
      protocol: TCP
    resources:
      requests:
        cpu: 200m
    startupProbe:
      failureThreshold: 24
      httpGet:
        host: 127.0.0.1
        path: /healthz
        port: probe-port
        scheme: HTTPS
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 15
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      name: flexvolume-dir
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
    - mountPath: /etc/kubernetes/controller-manager.conf
      name: kubeconfig
      readOnly: true
    - mountPath: /usr/local/share/ca-certificates
      name: usr-local-share-ca-certificates
      readOnly: true
    - mountPath: /usr/share/ca-certificates
      name: usr-share-ca-certificates
      readOnly: true
  hostNetwork: true
  priority: 2000001000
  priorityClassName: system-node-critical
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      type: DirectoryOrCreate
    name: flexvolume-dir
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
  - hostPath:
      path: /etc/kubernetes/controller-manager.conf
      type: FileOrCreate
    name: kubeconfig
  - hostPath:
      path: /usr/local/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-local-share-ca-certificates
  - hostPath:
      path: /usr/share/ca-certificates
      type: DirectoryOrCreate
    name: usr-share-ca-certificates
status: {}

## Si todo está bien, el deploy app llevará tres replicas
kubectl get deployments.apps -A -o wide 
NAMESPACE     NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
default       app       3/3     3            3           31m   nginx        nginx:alpine                              app=app
kube-system   coredns   2/2     2            2           38m   coredns      registry.k8s.io/coredns/coredns:v1.10.1   k8s-app=kube-dns
