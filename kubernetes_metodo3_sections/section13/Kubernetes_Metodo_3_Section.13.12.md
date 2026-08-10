## 115.1º Introducción a los componentes
- Los componentes ofrecen la habilidad de definir piezas de configuración lógica, que pueden incluir en multiples overlays
- Los componentes son útiles en situaciones donde las aplicaciones soportan multiples opciones que necesitan activarse solo debajo de los overlays

## 115.2º En la aplicación que se quiere desplegar lleva esta estructura. Los directorios ["Premium","Self_Hosted"] pertenecen al Caching, mientras los directorios ["Dev","Premium"] pertenecen al External DB
/root/code/k8s/
└── raiz_de_app
    ├── base
    ├── dev
    ├── Premium
    └── Self_Hosted

## 115.3º Si se realizan cambios en este tipo de aplicaciones, necesitarás cambiar los demás componentes para que funcionen bien. 
└── raiz_de_app
    ├── base
    ├── dev         --> ["DB"]
    ├── Premium     --> ["Caching","DB"]
    └── Self_Hosted --> ["Caching"]

## 113.4º Esta estructura incorpora el subdirectorio de componentes donde se establece las funciones de la aplicación, también está el overlays, caca uno tiene un fichero de kustomización 
/root/code/k8s/
├── base/
|   ├── kustomization.yaml
|   └── api-depl.yaml
├── components/
|   ├── caching/
|   |   ├── kustomization.yaml
|   |   ├── deployment-patch.yaml
|   |   └── redis-depl.yaml
|   └── db/
|       ├── kustomization.yaml
|       ├── deployment-patch.yaml
|       └── postgres-depl.yaml
└── overlays
    ├── dev/
    |   └── kustomization.yaml
    ├── premium/
    |   └── kustomization.yaml
    └── standalone/
        └── kustomization.yaml
