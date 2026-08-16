##
kubectl get nodes -o wide 
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   Ready    control-plane   22m   v1.35.0   10.244.55.212    <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22
node01         Ready    <none>          22m   v1.35.0   10.244.197.222   <none>        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.7.22

##
kubectl get nodes -o json > /opt/outputs/nodes.json

##
kubectl get nodes node01 -o json > /opt/outputs/node01.json 

##
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt

##
kubectl get nodes -o=jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os.txt 

##
ls -l /root/
total 4
-rw-r--r-- 1 root root 1404 Aug  3 05:57 my-kube-config
-rw-r--r-- 1 root root    0 Aug  3 05:57 sample.yaml

##
kubectl config view --kubeconfig=/root/my-kube-config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS
  name: development
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS
  name: kubernetes-on-aws
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS
  name: production
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS
  name: test-cluster-1
contexts:
- context:
    cluster: kubernetes-on-aws
    user: aws-user
  name: aws-user@kubernetes-on-aws
- context:
    cluster: test-cluster-1
    user: dev-user
  name: research
- context:
    cluster: development
    user: test-user
  name: test-user@development
- context:
    cluster: production
    user: test-user
  name: test-user@production
current-context: test-user@development
kind: Config
users:
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key

## 
kubectl config view --kubeconfig=/root/my-kube-config -o=jsonpath='{.users[*].name}' > /opt/outputs/users.txt
 
##
kubectl get pv
NAME       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
pv-log-1   100Mi      RWX            Retain           Available                          <unset>                          19m
pv-log-2   200Mi      RWX            Retain           Available                          <unset>                          19m
pv-log-3   300Mi      RWX            Retain           Available                          <unset>                          19m
pv-log-4   40Mi       RWX            Retain           Available                          <unset>                          19m

##
kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt

##
kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt

##
cat /opt/outputs/storage-capacity-sorted.txt 
NAME       CAPACITY
pv-log-4   40Mi
pv-log-1   100Mi
pv-log-2   200Mi
pv-log-3   300Mi

##
kubectl config view --kubeconfig=/root/my-kube-config -o=jsonpath='{.contexts[0].context.user}' > /opt/outputs/aws-context-name
kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name

