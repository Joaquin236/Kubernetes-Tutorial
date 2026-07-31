## 92.1º El cluster maestro puede ser inutilizado por una incidencia pero los nodos y clusters trabajadores mantienen el sistema activo. Cunado un Pod se bloquea, todo lo que contiene está fuera de servicio. Las réplicas duplican el pod dañado para evitar una caida del acceso. Pero si el maestro está desactivado no lo garantiza replicar, el api_server tampoco responde y no se puede establecer los comandos. La solución es disponer de varios nodos maestros y cluster para reactivar el controlador de KubeAdmin.

## 92.2º Cuando hay varios cluster maestro activo, de duplica los componentes internos de administración. Los Clusteres están conectado a un balanceador de carga que canaliza el recurso de sistema. Las istancias se ejecutan en paralelo, alguas acciones pueden ser duplicadas. Para evitar la duplicación de las órdenes el programador no debe estar en paralelo con el sistema maestro.

## 92.3º El progrmador Lider es el que recibe y ejecuta órdenes y el secundario estará en reserva. Con un comando se puede seleccionar
kube-controller-manager --leader-elect true
kube-controller-manager --leader-elect-lease-duration 15s
kube-controller-manager --leader-elect-renew-deadline 10s
kube-controller-manager --leader-elect-retry-period 2s
wget -q --https-only "https://github.com/coreos/etcd/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
tar -xfv etcd-v3.3.9-linux-amd64.tar.gz
mv -v etcd-v3.3.9-linux-amd64/etcd/* /usr/local/bin
mkdir -vp /etc/etcd /var/lib/etcd
cp -v ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd

## 92.4º El ETCD se puede separar el conjunto del cluster interno. Cuando uno de ellos deja de funcionar la reduncia de servicios puede quedar afectado. Al separarlo reduce el riesgo de perder los pods, nodos y contenedores. 
cat /etc/systemd/system/kube-apiserver.service

etcdctl put name ["User_name"]
etcdctl get name
etcdctl get / --prefix --keys-only
