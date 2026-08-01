## Activar el avance de paquetes IPv4. Crear dos estancias para interactuar con el Controlplane & node01
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward=1
EOF

# Apply sysctl params without reboot

sudo sysctl --system

# verify that net.ipv4.ip_forward is set to 1 with:
sysctl net.ipv4.ip_forward

## Ejecutar los comandos para instalar el kubelet, kubeadm y kubectl en cada estancia
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

# To see the new version labels
sudo apt-cache madison kubeadm

sudo apt-get install -y kubelet=1.35.0-1.1 kubeadm=1.35.0-1.1 kubectl=1.35.0-1.1

sudo apt-mark hold kubelet kubeadm kubectl

## La versión instalada es la 1.35.0
kubectl get nodes
E0731 15:57:21.852776   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.853811   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.854630   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.856625   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
E0731 15:57:21.857438   30476 memcache.go:265] "Unhandled Error" err="couldn't get current server API group list: the server could not find the requested resource"
Error from server (NotFound): the server could not find the requested resource

## Verificar la ip del eth0@if
ip a | grep eth0
4: eth0@if196366: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default qlen 1000
    inet 10.244.243.230/32 scope global eth0

## Inicia el api server con el kubeadm
kubeadm init --apiserver-advertise-address=10.244.243.230 --pod-network-cidr=10.244.0.0/16

## Ejecuta los comandos para crear $HOME.kube/config
mkdir -pv $HOME/.kube
sudo cp -vi /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown -v $(id -u):$(id -g) $HOME/.kube/config

## Después debe aparecer el nodo
kubectl get nodes
NAME           STATUS     ROLES           AGE    VERSION
controlplane   NotReady   control-plane   117s   v1.35.0

## Genera el token, este comando se revela cuando kubeadm init es exitoso. Válido en el node01
kubeadm join 10.244.243.230:6443 --token rhqmcg.0qgwkrwkp7jqq37a \
        --discovery-token-ca-cert-hash sha256:dbe468f018e0878b981778ae3b92cc9da8d52932e4df50e6167363f5a1d18c18 

## Desde el controlplane se visutaliza los pods
kubectl get nodes -o wide
NAME           STATUS     ROLES           AGE     VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION     CONTAINER-RUNTIME
controlplane   NotReady   control-plane   5m45s   v1.35.0   10.244.243.230   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26
node01         NotReady   [none]          50s     v1.35.0   10.244.127.230   [none]        Ubuntu 22.04.5 LTS   6.8.0-90-generic   containerd://1.6.26

## descarga el kube-flannel desde el navegador del host
https://sourceforge.net/projects/flannel.mirror/files/v0.28.8/kube-flannel.yml/download

## Crea un fichero con el contenido del yaml
nano kube-flannel.yml
apiVersion: v1
kind: Namespace
metadata:
  labels:
    k8s-app: flannel
    pod-security.kubernetes.io/enforce: privileged
  name: kube-flannel
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: flannel
  name: flannel
  namespace: kube-flannel
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: flannel
  name: flannel
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: flannel
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-flannel
---
apiVersion: v1
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "EnableNFTables": false,
      "Backend": {
        "Type": "vxlan"
      }
    }
kind: ConfigMap
metadata:
  labels:
    app: flannel
    k8s-app: flannel
    tier: node
  name: kube-flannel-cfg
  namespace: kube-flannel
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: flannel
    k8s-app: flannel
    tier: node
  name: kube-flannel-ds
  namespace: kube-flannel
spec:
  selector:
    matchLabels:
      app: flannel
      k8s-app: flannel
  template:
    metadata:
      labels:
        app: flannel
        k8s-app: flannel
        tier: node
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      containers:
      - args:
        - --ip-masq
        - --kube-subnet-mgr
        command:
        - /opt/bin/flanneld
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: EVENT_QUEUE_DEPTH
          value: "5000"
        - name: CONT_WHEN_CACHE_NOT_READY
          value: "false"
        image: ghcr.io/flannel-io/flannel:v0.28.8
        name: kube-flannel
        resources:
          requests:
            cpu: 100m
            memory: 50Mi
        securityContext:
          capabilities:
            add:
            - NET_ADMIN
            - NET_RAW
          privileged: false
        volumeMounts:
        - mountPath: /run/flannel
          name: run
        - mountPath: /etc/kube-flannel/
          name: flannel-cfg
        - mountPath: /run/xtables.lock
          name: xtables-lock
      hostNetwork: true
      initContainers:
      - args:
        - -f
        - /flannel
        - /opt/cni/bin/flannel
        command:
        - cp
        image: ghcr.io/flannel-io/flannel-cni-plugin:v1.9.1-flannel2
        name: install-cni-plugin
        volumeMounts:
        - mountPath: /opt/cni/bin
          name: cni-plugin
      - args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        command:
        - cp
        image: ghcr.io/flannel-io/flannel:v0.28.8
        name: install-cni
        volumeMounts:
        - mountPath: /etc/cni/net.d
          name: cni
        - mountPath: /etc/kube-flannel/
          name: flannel-cfg
      priorityClassName: system-node-critical
      serviceAccountName: flannel
      tolerations:
      - effect: NoSchedule
        operator: Exists
      volumes:
      - hostPath:
          path: /run/flannel
        name: run
      - hostPath:
          path: /opt/cni/bin
        name: cni-plugin
      - hostPath:
          path: /etc/cni/net.d
        name: cni
      - configMap:
          name: kube-flannel-cfg
        name: flannel-cfg
      - hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
        name: xtables-lock

## Edita el fichero para adaptarlo desde la sección del argumento
      - args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth0 
## Aplicar cambios
kubectl apply -f kube-flannel.yml 
namespace/kube-flannel created
serviceaccount/flannel created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds created

## Consultar los pods del espacio de nombres kube-flannel
kubectl get pods -n kube-flannel
NAME                    READY   STATUS                  RESTARTS   AGE
kube-flannel-ds-7lsjs   0/1     Init:ImagePullBackOff   0          5m5s
kube-flannel-ds-pf55d   0/1     Init:ImagePullBackOff   0          5m5s

## Cambiar el ns
kubectl config set-context --current --namespace=kube-flannel
Context "kubernetes-admin@kubernetes" modified.

## Consultar los pods del ns principal tras cambiar a kube-flannel
kubectl get pods
NAME                    READY   STATUS                  RESTARTS   AGE
kube-flannel-ds-7lsjs   0/1     Init:ImagePullBackOff   0          10m
kube-flannel-ds-pf55d   0/1     Init:ImagePullBackOff   0          10m

## Consutlar el log de un pod fuera de servicio
kubectl logs kube-flannel-ds-pf55d
Defaulted container "kube-flannel" out of: kube-flannel, install-cni-plugin (init), install-cni (init)
Error from server (BadRequest): container "kube-flannel" in pod "kube-flannel-ds-pf55d" is waiting to start: PodInitializing
