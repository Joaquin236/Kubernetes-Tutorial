## 117º El controlplane del cluster también puede fallar y necesitamos entender los fallos que puede sufrir. Algunos de los comandos son
kubectl get pods -A
service kube-system
kubectl get nodes
kubectl logs kube-apiserver -A
sudo journalctl -u kube-apiserver
