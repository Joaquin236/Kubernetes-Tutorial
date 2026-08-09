## 112.1º Estas son las opciones válidas que se pueden realizar con los parches en Kubernetes
- Los parches de Kustomize ofrece otros métodos de modificar la configuración de Kubernetes.
- Los parches se relizan aproximandose a un objetivo específico en las secciones del recurso de Kubernetes.
- Para crear el parche debes disponer de 3 parámetros:
  - Tipo de operación: ["añadir","borrar","reemplazar"]
  - Objetivo: El recurso que vamos a procesar:
    - Kind
    - Versión/Grupo
    - Nombre
    - Espacio de Nombre
    - labelSelector
    - AnnotationSelector
  - Valor: El contenido que se va a añadir, borrar o reemplazar

## 112.2º El fichero de los parches está ubicado en los ficheros kustomization.yaml que solemos crear manualmente, se declara el objetivo que debe apuntar y el atributo, en las opciones elegimos la petición y los valores nuevos.

## 112.3º Muestra de un despliegue
nano api-depl_1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: nginx
          image: nginx

## 112.4º Muestra del fichero de customización con el parche
nano kustomization-1.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        patch: /metadata/name
        value: web-deployment

## 112.5º El fichero cambia el nombre del despliegue sin editarlo manualmente. El valor nuevo es --> ["metadata.name:_web-deployment"]

## 112.6º Si queremos aumentar el número de réplicas con este fichero de customización, cambiamos el parámetro de objetivos y el nuevo valor de replicas
nano kustomization-2.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        patch: /spec/replicas
        value: 5

## 112.7º El valor nuevo es --> ["spec.replicas:_5"]

## 112.8º Las dos formas de expresar el conenido de un parche son: ["Json_6902_Patch","Strategic_Merge_Patch"]. La documentación para consultar es: ["https://datatracker.ietf.org/doc/html/rfc6902"] El el Json6902 se establece el valor del objetivo y el contenido de reemplazo, en el merge solo se establece el nombre del despliegue y el valor de reemplazo
nano kustomization-Json6902Patch_Inline_1.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        patch: /spec/replicas
        value: 5
------------------------------------------------
nano kustomization-Strategic_Merge_Patch_Inline_1.yaml
patches:
  - patch:
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: api-deployment
      spec:
        replicas: 5

## 112.9º Los dos ficheros de customización declarados están realizados con la estructura ["inline:Todo_en_un_solo_fichero"]. Existe otro método de estructura reutilizable con dos ficheros, el fichero kustomization.yaml guarda la ruta del segundo fichero y se vincula a los parámetros a modificar
nano kustomization-Json6902Patch_Separate_file_1.yaml
patches:
  - path: replica-patch-Json6902Patch_Values_1.yaml
    target:
      kind: Deployment
      name: nginx-deployment
-----------------------------------------------------
nano replica-patch-Json6902Patch_Values_1.yaml
- op: replace
  path: /spec/replicas
  value: 5
#################################################
nano kustomization-Strategic_Merge_Patch_Separate_file_1.yaml
patches:
  - replica-patch-Strategic-Merge-Patch_Values_1.yaml
-----------------------------------------------------
nano replica-patch-Strategic-Merge-Patch_Values_1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 5

## 112.10º A parte de modificar los nombres, la replica y la imagen también modifica el template y el valor de la API. 
nano nano kustomization-Json6902Patch_Inline_2.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/template/metadata/labels/component
        value: web

## 112.11º Al aplicar este fichero la ruta: ["spec.template.metadata.labels.component"] cambia el valor de origen por el nuevo valor
nano kustomization-Strategic_Merge_Patch_Separate_file_2.yaml
patches:
  - replica-patch-Strategic-Merge-Patch_Values_2.yaml
-----------------------------------------------------
nano replica-patch-Strategic-Merge-Patch_Values_2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        component: web

## 112.12º Si vamos a añadir el nombre de una organización al fichero de despliegue, se puede realizar con la opción add y la ruta: ["spec.template.metadata.labels.org"]
nano nano kustomization-Json6902Patch_Inline_3.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: add
        path: /spec/template/metadata/labels/org
        value: KodeKloud

## 112.13º Al aplicarlo, se añade el nombre de la organización asociado al despliegue de contenedores

## 112.14º El proceso se puede interpretar con el modo merge para redirigir el contenido en otro fichero
nano kustomization-Strategic_Merge_Patch_Separate_file_3.yaml
patches:
  - replica-patch-Strategic-Merge-Patch_Values_3.yaml
-----------------------------------------------------
nano replica-patch-Strategic-Merge-Patch_Values_3.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: KodeKloud

## 112.15º Para borrar un atributo usamos las mismas estructuras pero con la opción de borrar. Con solo establecer la ruta se completa el fichero
nano nano kustomization-Json6902Patch_Inline_4.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: remove
        path: /spec/template/metadata/labels/org

## 112.16º Para borrar usando este método sustituimos el valor de la org por null
nano kustomization-Strategic_Merge_Patch_Separate_file_4.yaml
patches:
  - replica-patch-Strategic-Merge-Patch_Values_4.yaml
-----------------------------------------------------
nano replica-patch-Strategic-Merge-Patch_Values_4.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: null

## 113.1º Las operaciones realizadas con el fichero de diccionario también son aplicables en una lista
nano kustomization_List-1.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        patch: /spec/template/spec/containers/0
        value: 
          name: haproxy
          image: haproxy

## 113.2º Para este fichero, se realiza cambios en la imagen de los contenedores, en lugar de llevar Nginx llevará el Haproxy. En el path del fichero debe apuntar a un índice donde se guarda el nombre y la imagen

## 113.3º Esta es la estructura para el modo merge del modo lista. Redirige el valor a otro fichero reutilizable
nano kustomization-Strategic_Merge_Patch_Separate_file_List-1.yaml
patches:
  - replica-patch-Strategic-Merge-Patch_Values_List-1.yaml
-----------------------------------------------------
nano replica-patch-Strategic-Merge-Patch_Values_List-1.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: haproxy

## 113.4º Para añadir más contenedores y generar una co-existencia usamos la operación ["add"] y la ruta acaba en ["-"]. La operación crea la nueva imagen debajo de la imagen inicial creando dos contenedores
nano kustomization_List-2.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: add
        patch: /spec/template/spec/containers/-
        value: 
          name: haproxy
          image: haproxy

## 113.5º Para reemplazar con el modo strategic merge usamos
nano kustomization-Strategic_Merge_Patch_Separate_file_List-2.yaml
patches:
  - replica-patch-Strategic-Merge-Patch_Values_List-2.yaml
-----------------------------------------------------
nano replica-patch-Strategic-Merge-Patch_Values_List-2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - name: haproxy
          image: haproxy

## 113.6º En un fichero de despliegue tenemos un contenedor de bases de datos que queremos borrar del fichero, necesitamos usar el índice y marcar la opción de borrado
nano api-depl_2.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      component: api
  template:
    metadata:
      labels:
        component: api
    spec:
      containers:
        - name: web
          image: nginx
        - name: database
          image: mongo
------------------------------------------------
nano kustomization_List_3.yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: remove
        path: /spec/template/spec/containers/1

## 113.7º Al aplicarlo borrará el índice con la imagen de la base de datos

## 113.8º El modo de borrar con el statregic merge necesita establecer el parámetro de borrado y el nombre de contendor
nano kustomization-Strategic_Merge_Patch_Separate_file_List-3.yaml
patches:
  - replica-patch-Strategic-Merge-Patch_Values_List-3.yaml
-----------------------------------------------------
nano replica-patch-Strategic-Merge-Patch_Values_List-3.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
        - $patch: delete
          name: database