## Consultar el fichero /var/lib/kubelet/config.yaml y filtrar por el containerRuntimeEndpoint
cat /var/lib/kubelet/config.yaml | grep unix
containerRuntimeEndpoint: unix:///var/run/containerd/containerd.sock

## Consulta la ruta de los ficheros binarios del CNI
ls -l /opt/cni/bin/
total 98452
-rwxr-xr-x 1 root root  5042186 Dec 18  2025 bandwidth
-rwxr-xr-x 1 root root  5694189 Dec 18  2025 bridge
-rwxr-xr-x 1 root root 13719696 Dec 18  2025 dhcp
-rwxr-xr-x 1 root root  5251247 Dec 18  2025 dummy
-rwxr-xr-x 1 root root  5701763 Dec 18  2025 firewall
-rwxr-xr-x 1 root root  2414517 Jul 25 17:22 flannel
-rwxr-xr-x 1 root root  5159307 Dec 18  2025 host-device
-rwxr-xr-x 1 root root  4350430 Dec 18  2025 host-local
-rwxr-xr-x 1 root root  5273398 Dec 18  2025 ipvlan
-rw-r--r-- 1 root root    11357 Dec 18  2025 LICENSE
-rwxr-xr-x 1 root root  4301450 Dec 18  2025 loopback
-rwxr-xr-x 1 root root  5306499 Dec 18  2025 macvlan
-rwxr-xr-x 1 root root  5107586 Dec 18  2025 portmap
-rwxr-xr-x 1 root root  5474778 Dec 18  2025 ptp
-rw-r--r-- 1 root root     2343 Dec 18  2025 README.md
-rwxr-xr-x 1 root root  4521078 Dec 18  2025 sbr
-rwxr-xr-x 1 root root  3772408 Dec 18  2025 static
-rwxr-xr-x 1 root root  5330851 Dec 18  2025 tap
-rwxr-xr-x 1 root root  4384728 Dec 18  2025 tuning
-rwxr-xr-x 1 root root  5266939 Dec 18  2025 vlan
-rwxr-xr-x 1 root root  4684912 Dec 18  2025 vrf

## Localizar el fichero de la ruta /etc/cni/net.d/*
ls -l /etc/cni/net.d/
total 4
-rw-r--r-- 1 root root 292 Jul 25 17:22 10-flannel.conflist

## Consulta el interior del fichero /etc/cni/net.d/10-flannel.conflist 
cat /etc/cni/net.d/10-flannel.conflist 
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