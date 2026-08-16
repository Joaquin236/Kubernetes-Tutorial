## Añade el acceso al kubeadm 1.35
nano /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /

## Aplica esta serie de comandos
kubectl drain controlplane --ignore-daemonsets ; apt update ; apt-cache madison kubeadm ; apt-get install kubeadm=1.35.0-1.1 ; kubeadm upgrade plan v1.35.0 ; kubeadm upgrade apply v1.35.0 ; apt-get install kubelet=1.35.0-1.1 ; systemctl daemon-reload ; systemctl restart kubelet ; kubectl uncordon controlplane

# Identify the taint first. 
kubectl describe node controlplane | grep -i taint

# Remove the taint with help of "kubectl taint" command.
kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-

# Verify it, the taint has been removed successfully.  
kubectl describe node controlplane | grep -i taint

## Entra en el nodo node01 para repetir el proceso
ssh node01

## Aplica esta serie de comandos después de verificar el fichero --> /etc/apt/sources.list.d/kubernetes.list
kubectl drain node01 --ignore-daemonsets ; apt update ; apt-cache madison kubeadm ; apt-get install kubeadm=1.35.0-1.1 ; kubeadm upgrade plan v1.35.0 ; kubeadm upgrade apply v1.35.0 ; apt-get install kubelet=1.35.0-1.1 ; systemctl daemon-reload ; systemctl restart kubelet ; kubectl uncordon controlplane

# Upgrade the node 
kubeadm upgrade node ; kubectl uncordon node01
kubectl get pods -o wide | grep gold # make sure this is scheduled on a node

## Realiza una consulta personalizada para obtener un informa con el sistema yaml
kubectl -n admin2406 get deployment -o custom-columns=DEPLOYMENT:.metadata.name,CONTAINER_IMAGE:.spec.template.spec.containers[].image,READY_REPLICAS:.status.readyReplicas,NAMESPACE:.metadata.namespace --sort-by=.metadata.name > /opt/admin2406_data
DEPLOYMENT   CONTAINER_IMAGE   READY_REPLICAS   NAMESPACE
deploy1      nginx             1                admin2406
deploy2      nginx:alpine      1                admin2406
deploy3      nginx:1.16        1                admin2406
deploy4      nginx:1.17        1                admin2406
deploy5      nginx:latest      1                admin2406

## El fichero /root/CKA/admin.kubeconfig tiene el puerto mal ajustado, editalo para corregirlo
Make sure the port for the kube-apiserver is correct. So for this change port from 4380 to 6443.
Run the below command to know the cluster information:
kubectl cluster-info --kubeconfig /root/CKA/admin.kubeconfig

## Crea un deploy con estos valores, después actualízalo a nginx:1.17
nano nginx-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-deploy
  template:
    metadata: 
      labels:
        app: nginx-deploy
    spec:
      containers:
        - name: nginx
          image: "nginx:1.16"
## Aplica el deploy y los parches de imagen          
kubectl apply -f /root/nginx-deploy.yaml
kubectl set image deployment/nginx-deploy nginx=nginx:1.17 ; kubectl annotate deployment nginx-deploy kubernetes.io/change-cause="Updated nginx image to 1.17"
deployment.apps/nginx-deploy configured
deployment.apps/nginx-deploy image updated
deployment.apps/nginx-deploy annotated

kubectl get deployments.apps nginx-deploy -o wide 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
nginx-deploy   1/1     1            1           96s   nginx        nginx:1.17   app=nginx-deploy

##
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-alpha-pvc
  namespace: alpha
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: slow

## Crea una copia de seguridad del certificado
export ETCDCTL_API=3
etcdctl snapshot save --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --endpoints=127.0.0.1:2379 /opt/etcd-backup.db

## Crea un pod con estos valores con formato de comando
kubectl run secret-1401 -n admin1401 --image=busybox --dry-run=client -oyaml --command -- sleep 4800 > admin.yaml

## Crea un pod con estos valores en formato yaml
---
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: secret-1401
  name: secret-1401
  namespace: admin1401
spec:
  volumes:
  - name: secret-volume
    # secret volume
    secret:
      secretName: dotfile-secret
  containers:
  - command:
    - sleep
    - "4800"
    image: busybox
    name: secret-admin
    # volumes' mount path
    volumeMounts:
    - name: secret-volume
      readOnly: true
      mountPath: "/etc/secret-volume"

