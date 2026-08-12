## 118º De vez en cuando puede que algún nodo se desconecte y esté con el estado de no-preparado y no puede interactuar con el cluster. Hay opciones de monitorizarlo con el comando
kubectl get nodes
kubectl describe node
top # --> task manager Linux CUI
df -f
service kubelet status
sudo journalctl -u kubelet
openssl x509 -in /var/lib/kubelet/worker-1.crt -text
