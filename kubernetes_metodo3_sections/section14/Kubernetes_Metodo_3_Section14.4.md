##
sudo journalctl -u kubelet.service 
Aug 12 12:39:10 controlplane kubelet[1921]: E0812 12:39:10.937320 # --> failed to load Kubelet config file /var/lib/kubelet/config.yaml

##
kubectl get pods -n kube-system -o wide 
NAME                                   READY   STATUS             RESTARTS      AGE     IP              NODE           NOMINATED NODE   READINESS GATES
coredns-6f6c7df987-9pxj8               1/1     Running            0             13m     172.17.0.3      controlplane   <none>           <none>
coredns-6f6c7df987-qwmmt               1/1     Running            0             13m     172.17.0.2      controlplane   <none>           <none>
etcd-controlplane                      1/1     Running            0             13m     10.244.170.83   controlplane   <none>           <none>
kube-apiserver-controlplane            1/1     Running            0             13m     10.244.170.83   controlplane   <none>           <none>
kube-controller-manager-controlplane   1/1     Running            0             13m     10.244.170.83   controlplane   <none>           <none>
kube-proxy-mb4tn                       1/1     Running            0             13m     10.244.170.83   controlplane   <none>           <none>
kube-scheduler-controlplane            0/1     CrashLoopBackOff   6 (42s ago)   6m26s   10.244.170.83   controlplane   <none>           <none>

##
kubectl get componentstatuses -o wide -A
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                        ERROR
scheduler            Unhealthy   Get "https://127.0.0.1:10259/healthz": dial tcp 127.0.0.1:10259: connect: connection refused   
controller-manager   Healthy     ok                                                                                             
etcd-0               Healthy     ok     

##
kubectl logs -n kube-system kube-scheduler-controlplane # --> There aren't logs at the pod

##
kubectl describe pods -n kube-system kube-scheduler-controlplane | grep -A5 Events
Events:
  Type     Reason   Age                   From     Message
  ----     ------   ----                  ----     -------
  Warning  BackOff  8m13s (x19 over 12m)  kubelet  spec.containers{kube-scheduler}: Back-off restarting failed container kube-scheduler in pod kube-scheduler-controlplane_kube-system(186c52b5584d78a99894b5f06124bf5d)
  Normal   Pulled   114s (x8 over 12m)    kubelet  spec.containers{kube-scheduler}: Container image "registry.k8s.io/kube-scheduler:v1.35.0" already present on machine and can be accessed by the pod
  Normal   Created  114s (x8 over 12m)    kubelet  spec.containers{kube-scheduler}: Container created

##
kubectl get pods -n kube-system kube-scheduler -o yaml | grep -A1 -B2 command
spec:
  containers:
  - command:
    - kube-schedulerrrr

##
kubectl get pod -n kube-system kube-scheduler-controlplane -o yaml > kube-scheduler_file.yaml

##
nano kube-scheduler_file.yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/config.hash: 186c52b5584d78a99894b5f06124bf5d
    kubernetes.io/config.mirror: 186c52b5584d78a99894b5f06124bf5d
    kubernetes.io/config.seen: "2026-08-12T12:46:12.877284997Z"
    kubernetes.io/config.source: file
  creationTimestamp: "2026-08-12T12:46:24Z"
  generation: 1
  labels:
    component: kube-scheduler
    tier: control-plane
  name: kube-scheduler-controlplane
  namespace: kube-system
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node
    name: controlplane
    uid: bee24793-c72a-471e-87c9-c88323555a3c
  resourceVersion: "1574"
  uid: 4701d818-a09c-4c04-a570-9238477b40a0
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
      successThreshold: 1
      timeoutSeconds: 15
    name: kube-scheduler
    ports:
    - containerPort: 10259
      hostPort: 10259
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
      successThreshold: 1
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
      successThreshold: 1
      timeoutSeconds: 15
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  hostNetwork: true
  nodeName: controlplane
  preemptionPolicy: PreemptLowerPriority
  priority: 2000001000
  priorityClassName: system-node-critical
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    operator: Exists
  volumes:
  - hostPath:
      path: /etc/kubernetes/scheduler.conf
      type: FileOrCreate
    name: kubeconfig
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T12:46:24Z"
    status: "True"
    type: PodReadyToStartContainers
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T12:39:18Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T12:46:24Z"
    message: 'containers with unready status: [kube-scheduler]'
    reason: ContainersNotReady
    status: "False"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T12:46:24Z"
    message: 'containers with unready status: [kube-scheduler]'
    reason: ContainersNotReady
    status: "False"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2026-08-12T12:39:18Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - allocatedResources:
      cpu: 100m
    containerID: containerd://1ed30c173374ef152d27df65813421ad947fd5070d1de94945254ae26556707d
    image: registry.k8s.io/kube-scheduler:v1.35.0
    imageID: registry.k8s.io/kube-scheduler@sha256:0ab622491a82532e01876d55e365c08c5bac01bcd5444a8ed58c1127ab47819f
    lastState:
      terminated:
        containerID: containerd://1ed30c173374ef152d27df65813421ad947fd5070d1de94945254ae26556707d
        exitCode: 128
        finishedAt: "2026-08-12T12:57:12Z"
        message: 'failed to create containerd task: failed to create shim task: OCI
          runtime create failed: runc create failed: unable to start container process:
          exec: "kube-schedulerrrr": executable file not found in $PATH: unknown'
        reason: StartError
        startedAt: "1970-01-01T00:00:00Z"
    name: kube-scheduler
    ready: false
    resources:
      requests:
        cpu: 100m
    restartCount: 7
    started: false
    state:
      waiting:
        message: back-off 5m0s restarting failed container=kube-scheduler pod=kube-scheduler-controlplane_kube-system(186c52b5584d78a99894b5f06124bf5d)
        reason: CrashLoopBackOff
    volumeMounts:
    - mountPath: /etc/kubernetes/scheduler.conf
      name: kubeconfig
      readOnly: true
      recursiveReadOnly: Disabled
  hostIP: 10.244.170.83
  hostIPs:
  - ip: 10.244.170.83
  phase: Running
  podIP: 10.244.170.83
  podIPs:
  - ip: 10.244.170.83
  qosClass: Burstable
  startTime: "2026-08-12T12:39:18Z"

##
kubectl delete pods -n kube-system kube-scheduler-controlplane 
pod "kube-scheduler-controlplane" deleted from kube-system namespace
(Borrar este pod no sirve, vuelve a crear el mismo con el valor defectuoso)

##
kubectl get deploy -A -o wide 
NAMESPACE     NAME      READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                    SELECTOR
default       app       0/1     1            0           36m   nginx        nginx:alpine                              app=app
kube-system   coredns   2/2     2            2           43m   coredns      registry.k8s.io/coredns/coredns:v1.10.1   k8s-app=kube-dns

##
kubectl get deployments.apps app -o yaml > app_file.yaml

##
nano app_file.yaml

##
kubectl apply -f app_file.yaml 
Warning: resource deployments/app is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.
deployment.apps/app configured

##
kubectl get pods -n kube-system 
NAME                                   READY   STATUS             RESTARTS        AGE
coredns-6f6c7df987-9pxj8               1/1     Running            0               47m
coredns-6f6c7df987-qwmmt               1/1     Running            0               47m
etcd-controlplane                      1/1     Running            0               47m
kube-apiserver-controlplane            1/1     Running            0               47m
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   5 (116s ago)    4m44s
kube-proxy-mb4tn                       1/1     Running            0               47m
kube-scheduler-controlplane            0/1     CrashLoopBackOff   12 (4m1s ago)   7m56s

##
kubectl logs -n kube-system kube-controller-manager-controlplane 
I0812 13:27:40.375955       1 serving.go:386] Generated self-signed cert in-memory
E0812 13:27:40.376038       1 run.go:72] "command failed" err="stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory"

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

##
kubectl get pods -n kube-system 
NAME                                   READY   STATUS             RESTARTS       AGE
coredns-6f6c7df987-9pxj8               1/1     Running            0              54m
coredns-6f6c7df987-qwmmt               1/1     Running            0              54m
etcd-controlplane                      1/1     Running            0              54m
kube-apiserver-controlplane            1/1     Running            0              54m
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   3 (16s ago)    53s
kube-proxy-mb4tn                       1/1     Running            0              54m
kube-scheduler-controlplane            0/1     CrashLoopBackOff   14 (51s ago)   14m

##
kubectl logs -n kube-system kube-controller-manager-controlplane 
I0812 13:33:27.413094       1 serving.go:386] Generated self-signed cert in-memory
E0812 13:33:27.414590       1 run.go:72] "command failed" err="unable to load client CA provider: open /etc/kubernetes/pki/ca.crt: no such file or directory"

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
