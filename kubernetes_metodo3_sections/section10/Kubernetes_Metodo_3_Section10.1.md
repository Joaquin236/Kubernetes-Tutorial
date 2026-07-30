## 91.1º Propósitos que debe cumplir el diseño de un cluster: ["Diseño_Educativo","Desarrollo_de_aplicaciones","Alojar_servicios_y_aplicaciones_web"]. ["Alojado_en_la_nube","Bajo_Demanda"]. ["Despliegue_de_servicios_web","Despliegue_de_Investigación"]. ["Uso_intensivo_de_CPU","Uso_intensivo_de_RAM"]. ["Tráfico_pesado_intensivo","Tráfico_con_ráfafas"]

## 91.2º Las aplicaciones para realizar el modo aprendizaje son: ["Minikube","Single_Cluster_KubeAdm/GCP/AWS"] Para el entorno de pruebas: ["Multi_Cluster_GCP/AWS/AKS"]

## 91.3º Para el despliegue de una aplciación de producción: ["Alta_disponibilidad_de_cluster_multi-nodos","KubeAdm/GCP/AWS","Puede_superar_los_5.000_nodos","Puede_superar_los_150.000_pods_en_el_cluster","Puede_superar_los_300.000_Contenedores","Puede_superar_los_100_Pods_por_nodo"]

## 91.4º Tabla detallada de la cantidad de nodos entre GCP/AWS
|Nodes  |GCP_Column_1   |GCP_Column_2           |AWS_Column_1   |AWS_Column_2            |
|-------|---------------|-----------------------|---------------|------------------------|
|1-5    |N1-Standard-1  |1  vCPU  + 3,75 GB  RAM|M3.medium      |1  vCPU  + 3,75 GB RAM  |
|6-10   |N1-Standard-2  |2  vCPU  + 7,50 GB  RAM|M3.large       |2  vCPU  + 7,50 GB RAM  |
|11-100 |N1-Standard-4  |4  vCPU  + 15,0 GB  RAM|M3.xlarge      |4  vCPU  + 15,0 GB RAM  |
|101-250|N1-Standard-8  |8  vCPU  + 30,0 GB  RAM|M3.2xlarge     |8  vCPU  + 30,0 GB RAM  |
|251-500|N1-Standard-16 |16 vCPU  + 60,0 GB  RAM|C4.4xlarge     |16 vCPU  + 30,0 GB RAM  |
|>500   |N1-Standard-32 |32 vCPU  + 120  GB  RAM|C4.8xlarge     |36 vCPU  + 60,0 GB RAM  |

## 91.5º Para elegir entre conexión por la nube o bajo demamda. Neceistamos evaluar si queremos usar ["Kubeadm_para_on-prem","GKE_para_GCP","EKS_para_AWS","AKS_para_Azure"].

## 91.6º Para elegir el almacenamiento del alojamiento, se necesita evaluar ["SSD-Backend_Alto-Rendimiento","Conexión_Multiple","Compatir_volumenes_al_pod","Etiquetar_nodos","Seleccionar_nodos"]
