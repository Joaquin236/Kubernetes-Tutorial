## Consultar los nodos, el node01 está fuera de servicio
kubectl get nodes -o wide 
NAME           STATUS     ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready      control-plane   14m   v1.35.0   10.244.105.129   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         NotReady   [none]          13m   v1.35.0   10.244.105.174   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Consulta el registro de error
journalctl -u kubelet.service 
Aug 12 16:35:57 controlplane kubelet[1934]: E0812 16:35:57.783931 --> failed to load Kubelet config file /var/lib/kubelet/config.yaml

## Describe el nodo y filtra por eventos
kubectl describe node node01 | grep -A6 Events 
Events:
  Type    Reason          Age    From             Message
  ----    ------          ----   ----             -------
  Normal  RegisteredNode  16m    node-controller  Node node01 event: Registered Node node01 in Controller
  Normal  NodeNotReady    2m58s  node-controller  Node node01 status is now: NodeNotReady

## Conectarse a la estancia ssh del node01 y reinicia el servicio
ssh node01
service kubelet start ; service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /usr/lib/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Wed 2026-08-12 16:55:39 UTC; 30s ago
       Docs: https://kubernetes.io/docs/
   Main PID: 11322 (kubelet)
      Tasks: 22 (limit: 75883)
     Memory: 27.6M
        CPU: 340ms
     CGroup: /system.slice/kubelet.service
             └─11322 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf

Aug 12 16:55:40 node01 kubelet[11322]: I0812 16:55:40.207963   11322 kubelet_node_status.go:123] "Node was previously
Aug 12 16:55:40 node01 kubelet[11322]: I0812 16:55:40.208029   11322 kubelet_node_status.go:77] "Successfully registered
Aug 12 16:55:40 node01 kubelet[11322]: I0812 16:55:40.946302   11322 apiserver.go:52] "Watching apiserver"
Aug 12 16:55:40 node01 kubelet[11322]: I0812 16:55:40.956684   11322 desired_state_of_world_populator.go:154] "Finished
Aug 12 16:55:41 node01 kubelet[11322]: I0812 16:55:41.004603   11322 reconciler_common.go:251] "operationExecutor
Aug 12 16:55:41 node01 kubelet[11322]: I0812 16:55:41.004643   11322 reconciler_common.go:251] "operationExecutor
Aug 12 16:55:41 node01 kubelet[11322]: I0812 16:55:41.004660   11322 reconciler_common.go:251] "operationExecutor
Aug 12 16:55:41 node01 kubelet[11322]: I0812 16:55:41.004682   11322 reconciler_common.go:251] "operationExecutor
Aug 12 16:55:41 node01 kubelet[11322]: I0812 16:55:41.004695   11322 reconciler_common.go:251] "operationExecutor
Aug 12 16:55:41 node01 kubelet[11322]: I0812 16:55:41.004708   11322 reconciler_common.go:251] "operationExecutor

## Vuelve a consultar el nodo
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   21m   v1.35.0   10.244.105.129   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    [none]          21m   v1.35.0   10.244.105.174   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

----------------------------------------------------------------------------------------------

## El nodo vuelve a fallar, vuelve a reactivarlo
journalctl -u kubelet.service -f
Aug 12 17:08:26 controlplane kubelet[3808]: E0812 17:08:26.214273    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/etcd-controlplane" containerName="etcd"
Aug 12 17:08:30 controlplane kubelet[3808]: E0812 17:08:30.211118    3808 prober_manager.go:209] "Readiness probe already exists for container" pod="kube-system/coredns-6f6c7df987-7hfmr" containerName="coredns"
Aug 12 17:08:35 controlplane kubelet[3808]: E0812 17:08:35.211305    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/kube-scheduler-controlplane" containerName="kube-scheduler"
Aug 12 17:08:37 controlplane kubelet[3808]: E0812 17:08:37.210746    3808 prober_manager.go:209] "Readiness probe already exists for container" pod="kube-system/coredns-6f6c7df987-p9wjr" containerName="coredns"
Aug 12 17:08:37 controlplane kubelet[3808]: E0812 17:08:37.210787    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/kube-controller-manager-controlplane" containerName="kube-controller-manager"
Aug 12 17:09:26 controlplane kubelet[3808]: E0812 17:09:26.211120    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/kube-apiserver-controlplane" containerName="kube-apiserver"

## Conecta con el node01 y edita el fichero, la línea del clientCAFile
ssh node01
nano /var/lib/kubelet/config.yaml 
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 0s
    cacheUnauthorizedTTL: 0s
cgroupDriver: systemd
clusterDNS:
- 172.20.0.10
clusterDomain: cluster.local
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock
cpuManagerReconcilePeriod: 0s
crashLoopBackOff: {}
evictionPressureTransitionPeriod: 0s
fileCheckFrequency: 0s
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 0s
imageMaximumGCAge: 0s
imageMinimumGCAge: 0s
kind: KubeletConfiguration
logging:
  flushFrequency: 0
  options:
    json:
      infoBufferSize: "0"
    text:
      infoBufferSize: "0"
  verbosity: 0
memorySwap: {}
nodeStatusReportFrequency: 0s
nodeStatusUpdateFrequency: 0s
rotateCertificates: true
runtimeRequestTimeout: 0s
shutdownGracePeriod: 0s
shutdownGracePeriodCriticalPods: 0s
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 0s
syncFrequency: 0s
volumeStatsAggPeriod: 0s

## El fichero corregido devuelve a su estado operarivo el node01
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   46m   v1.35.0   10.244.105.129   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    [none]          45m   v1.35.0   10.244.105.174   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

----------------------------------------------------------------------------------------------

## El nodo ha vuelto a fallar, vuelve a reactivarlo una vez más
kubectl get nodes -o wide 
NAME           STATUS     ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready      control-plane   48m   v1.35.0   10.244.105.129   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         NotReady   [none]          48m   v1.35.0   10.244.105.174   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

## Consultar el registro del kubelet
journalctl -u kubelet.service -f
Aug 12 17:23:00 controlplane kubelet[3808]: E0812 17:23:00.210949    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/kube-scheduler-controlplane" containerName="kube-scheduler"
Aug 12 17:23:19 controlplane kubelet[3808]: E0812 17:23:19.210874    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/kube-apiserver-controlplane" containerName="kube-apiserver"
Aug 12 17:23:33 controlplane kubelet[3808]: E0812 17:23:33.210659    3808 prober_manager.go:209] "Readiness probe already exists for container" pod="kube-system/coredns-6f6c7df987-p9wjr" containerName="coredns"
Aug 12 17:23:39 controlplane kubelet[3808]: E0812 17:23:39.210982    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/etcd-controlplane" containerName="etcd"
Aug 12 17:23:46 controlplane kubelet[3808]: E0812 17:23:46.210888    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/kube-controller-manager-controlplane" containerName="kube-controller-manager"
Aug 12 17:24:12 controlplane kubelet[3808]: E0812 17:24:12.210907    3808 prober_manager.go:197] "Startup probe already exists for container" pod="kube-system/kube-scheduler-controlplane" containerName="kube-scheduler"

## Entra con el node01, edita el fichero con el error, el puerto de servidor debe ser cambiado
ssh node01
nano /etc/kubernetes/kubelet.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ["LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJYW"]
    server: https://controlplane:6443
  name: default-cluster
contexts:
- context:
    cluster: default-cluster
    namespace: default
    user: default-auth
  name: default-context
current-context: default-context
kind: Config
users:
- name: default-auth
  user:
    client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
    client-key: /var/lib/kubelet/pki/kubelet-client-current.pem

## Reinicia el servicio para que se recupere
systemctl restart kubelet.service 

## Vueve a consultar el estdo del nodo
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   55m   v1.35.0   10.244.105.129   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    [none]          54m   v1.35.0   10.244.105.174   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
