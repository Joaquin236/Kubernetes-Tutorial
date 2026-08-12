## Consulta los pods y filtrar por estado
kubectl get pods -A | grep Pending
triton        mysql                                  0/1     Pending   0          49s
triton        webapp-mysql-58c8d859b-9lpjb           0/1     Pending   0          49s

## Obtener los nodos
kubectl get nodes
NAME           STATUS     ROLES           AGE   VERSION
controlplane   NotReady   control-plane   11m   v1.35.0

## Consulta el registro
journalctl -u kubelet | grep -i cni
Aug 12 16:38:41 controlplane kubelet[2791]: I0812 16:38:41.838225    2791 reconciler_common.go:251] "operationExecutor.VerifyControllerAttachedVolume started for volume \"cni-plugin\" (UniqueName: \"kubernetes.io/host-path/60d629b8-7d1e-46f7-993c-f1d27ddb5ae3-cni-plugin\") pod \"kube-flannel-ds-pqhtj\" (UID: \"60d629b8-7d1e-46f7-993c-f1d27ddb5ae3\") " pod="kube-flannel/kube-flannel-ds-pqhtj"
Aug 12 16:38:41 controlplane kubelet[2791]: I0812 16:38:41.838256    2791 reconciler_common.go:251] "operationExecutor.VerifyControllerAttachedVolume started for volume \"cni\" (UniqueName: \"kubernetes.io/host-path/60d629b8-7d1e-46f7-993c-f1d27ddb5ae3-cni\") pod \"kube-flannel-ds-pqhtj\" (UID: \"60d629b8-7d1e-46f7-993c-f1d27ddb5ae3\") " pod="kube-flannel/kube-flannel-ds-pqhtj"

## Consultar el directorio net.d
ls -lah /etc/cni/net.d/
total 8.0K
drwx------ 2 root root 4.0K Aug 12 16:48 .
drwxr-xr-x 3 root root 4.0K Feb 19 07:06 ..
-rw-r--r-- 1 root root    0 Dec 18  2025 .kubernetes-cni-keep

## Aplica el plugin de pod flannel
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel configured
clusterrolebinding.rbac.authorization.k8s.io/flannel unchanged
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

## Consulta del flannel
kubectl get pods -A | grep -E "flannel|calico|weave|cilium"

## Registro de eventos en el ns triton
kubectl describe pod -n triton mysql | grep -A6 Events
Events:
  Type     Reason            Age    From               Message
  ----     ------            ----   ----               -------
  Warning  FailedScheduling  6m10s  default-scheduler  0/1 nodes are available: 1 node(s) had untolerated taint(s). no new claims to deallocate, preemption: 0/1 nodes are available: 1 Preemption is not helpful for scheduling.
  Normal   Scheduled         86s    default-scheduler  Successfully assigned triton/mysql to controlplane
  Normal   Pulling           85s    kubelet            spec.containers{mysql}: Pulling image "mysql:5.6"
  Normal   Pulled            79s    kubelet            spec.containers{mysql}: Successfully pulled image "mysql:5.6" in 4.808s (6.467s including waiting). Image size: 102984033 bytes.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Localiza el pod con error
kubectl get pods -o wide -A | grep Error
kube-system    kube-proxy-cl7p5                       0/1     Error     2 (37s ago)   41s     10.244.88.17   controlplane   [none]           [none]

## Describe el pod kube-proxy
kubectl describe pods -n kube-system kube-proxy-cl7p5 | grep -A5 Event
Events:
  Type     Reason            Age                  From               Message
  ----     ------            ----                 ----               -------
  Normal   Scheduled         2m1s                 default-scheduler  Successfully assigned kube-system/kube-proxy-cl7p5 to controlplane
  Normal   Pulling           2m                   kubelet            spec.containers{kube-proxy}: Pulling image "registry.k8s.io/kube-proxy:v1.26.0"
  Normal   Pulled            118s                 kubelet            spec.containers{kube-proxy}: Successfully pulled image "registry.k8s.io/kube-proxy:v1.26.0" in 2.454s (2.454s including waiting). Image size: 21536465 bytes.

## Describe el kube-proxy
kubectl -n kube-system describe ds kube-proxy

## Consulta el log del kube-proxy
kubectl logs -n kube-system kube-proxy-cl7p5 
E0812 20:59:08.468690       1 run.go:74] "command failed" err="failed complete: open /var/lib/kube-proxy/configuration.conf: no such file or directory"

## Consulta el configmap en formato yaml
kubectl get configmap -n kube-system kube-proxy -o yaml

## Describe el kube-proxy
kubectl -n kube-system describe ds kube-proxy

## Edita el kube-proxy
kubectl -n kube-system edit ds kube-proxy

    spec:
      containers:
      - command:
        - /usr/local/bin/kube-proxy
        - --config=/var/lib/kube-proxy/configuration.conf # --> Replace for: ["- --config=/var/lib/kube-proxy/config.conf"]
        - --hostname-override=$(NODE_NAME)

## Edición completada
kubectl -n kube-system edit ds kube-proxy
daemonset.apps/kube-proxy edited

## Consulta si todos los pods están activos
kubectl get pods -n kube-system -o wide 
NAME                                   READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
coredns-54f54cff85-qgjg7               1/1     Running   0          28m   10.244.0.5     controlplane   [none]           [none]
coredns-54f54cff85-rz7zc               1/1     Running   0          28m   10.244.0.4     controlplane   [none]           [none]
etcd-controlplane                      1/1     Running   0          28m   10.244.88.17   controlplane   [none]           [none]
kube-apiserver-controlplane            1/1     Running   0          28m   10.244.88.17   controlplane   [none]           [none]
kube-controller-manager-controlplane   1/1     Running   0          28m   10.244.88.17   controlplane   [none]           [none]
kube-proxy-5sd7z                       1/1     Running   0          38s   10.244.88.17   controlplane   [none]           [none]
kube-scheduler-controlplane            1/1     Running   0          28m   10.244.88.17   controlplane   [none]           [none]
