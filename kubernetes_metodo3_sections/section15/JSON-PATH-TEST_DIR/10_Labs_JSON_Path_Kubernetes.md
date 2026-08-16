## ¿Cómo está realizado este fichero JSON?
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      }
    ],
    "nodeName": "node01"
  }
}
--> This JSON begin like dictionary

## ¿Cuantos objetos tiene este JSON?
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      }
    ],
    "nodeName": "node01"
  }
}
--> This JSON have 4 objects

## ¿Cómo está realizado este fichero JSON?
[
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-2",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node02"
    }
  }
]
--> This JSON begin like list

## ¿Cuantos elementos tiene este JSON?
[
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-2",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node02"
    }
  }
]
--> This JSON have 2 items

## ¿Que tipo de valor es el valor del apiVersion?
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      }
    ],
    "nodeName": "node01"
  }
}
--> The apiVersion_value of this JSON is a string

## ¿Que tipo de valor es el valor del metadata?
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      }
    ],
    "nodeName": "node01"
  }
}
--> The metadata field is a dictionary

## ¿Cómo está realizado este fichero JSON?
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      }
    ],
    "nodeName": "node01"
  }
}
--> This JSON have a list of dictionaries

## Contenido del fichero input.json
cat input.json 
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      }
    ],
    "nodeName": "node01"
  }
}

## Consulta el nombre de la clave kind y exporta el comando a un fichero
cat input.json | jpath $.kind
[
  "Pod"
]
echo "cat input.json | jpath $.kind" > /root/answer9.sh

## Consulta el nombre de la clave metadata.name y exporta el comando a un fichero
cat input.json | jpath $.metadata.name
[
  "nginx-pod"
]
echo "cat input.json | jpath $.metadata.name" > /root/answer10.sh

## Consulta el nombre de la clave spec.nodeName y exporta el comando a un fichero
cat input.json | jpath $.spec.nodeName
[
  "node01"
]
echo "cat input.json | jpath $.spec.nodeName" > /root/answer11.sh

## Consulta el nombre de la clave spec.containers y exporta el comando a un fichero
cat input.json | jpath $.spec.containers[0]
[
  {
    "image": "nginx:alpine",
    "name": "nginx"
  }
]
echo "cat input.json | jpath $.spec.containers[0]" > /root/answer12.sh

## Consulta el nombre de la clave spec.containers.image y exporta el comando a un fichero
cat input.json | jpath $.spec.containers[0].image
[
  "nginx:alpine"
]
echo "cat input.json | jpath $.spec.containers[0].image" > /root/answer13.sh

## Contenido del fichero k8status.json
cat k8status.json 
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      },
      {
        "image": "redis:alpine",
        "name": "redis-container"
      }
    ],
    "nodeName": "node01"
  },
  "status": {
    "conditions": [
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2019-06-13T05:34:09Z",
        "status": "True",
        "type": "Initialized"
      },
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2019-06-13T05:34:09Z",
        "status": "True",
        "type": "PodScheduled"
      }
    ],
    "containerStatuses": [
      {
        "image": "nginx:alpine",
        "name": "nginx",
        "ready": false,
        "restartCount": 4,
        "state": {
          "waiting": {
            "reason": "ContainerCreating"
          }
        }
      },
      {
        "image": "redis:alpine",
        "name": "redis-container",
        "ready": false,
        "restartCount": 2,
        "state": {
          "waiting": {
            "reason": "ContainerCreating"
          }
        }
      }
    ],
    "hostIP": "172.17.0.75",
    "phase": "Pending",
    "qosClass": "BestEffort",
    "startTime": "2019-06-13T05:34:09Z"
  }
}

## Consulta el nombre de la clave status.phase y exporta el comando a un fichero
cat k8status.json | jpath $.status.phase
[
  "Pending"
]
echo "cat k8status.json | jpath $.status.phase" > /root/answer14.sh

## Consulta el nombre de la clave containerStatuses.state.waiting.reason y exporta el comando a un fichero
cat k8status.json | jpath $.*.containerStatuses[0].state.waiting.reason
[
  "ContainerCreating"
]
echo "cat k8status.json | jpath $.*.containerStatuses[0].s
tate.waiting.reason" > /root/answer15.sh

## Consulta el nombre de la clave containerStatuses.restartCount y exporta el comando a un fichero
cat k8status.json | jpath $.*.containerStatuses[1].restartCount
[
  2
]
echo "cat k8status.json | jpath $.*.containerStatuses[1].restartCount" > /root/answer16.sh

## Contenido del fichero podslist.json
cat podslist.json 
[
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-2",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node02"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-3",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-4",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "db-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "mysql",
          "name": "mysql"
        }
      ],
      "nodeName": "node01"
    }
  }
]

## Consulta el nombre de la clave *.metadata.name y exporta el comando a un fichero
cat podslist.json | jpath $.*.metadata.name
[
  "web-pod-1",
  "web-pod-2",
  "web-pod-3",
  "web-pod-4",
  "db-pod-1"
]
echo "cat podslist.json | jpath $.*.metadata.name" > /root/answer17.sh

## Contenido del fichero userlist.json
cat userslist.json 
{
  "kind": "Config",
  "apiVersion": "v1",
  "preferences": {},
  "clusters": [
    {
      "name": "development",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    },
    {
      "name": "kubernetes-on-aws",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    },
    {
      "name": "production",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    },
    {
      "name": "test-cluster-1",
      "cluster": {
        "server": "KUBE_ADDRESS",
        "certificate-authority": "/etc/kubernetes/pki/ca.crt"
      }
    }
  ],
  "users": [
    {
      "name": "aws-user",
      "user": {
        "client-certificate": "/etc/kubernetes/pki/users/aws-user/aws-user.crt",
        "client-key": "/etc/kubernetes/pki/users/aws-user/aws-user.key"
      }
    },
    {
      "name": "dev-user",
      "user": {
        "client-certificate": "/etc/kubernetes/pki/users/dev-user/developer-user.crt",
        "client-key": "/etc/kubernetes/pki/users/dev-user/dev-user.key"
      }
    },
    {
      "name": "test-user",
      "user": {
        "client-certificate": "/etc/kubernetes/pki/users/test-user/test-user.crt",
        "client-key": "/etc/kubernetes/pki/users/test-user/test-user.key"
      }
    }
  ],
  "contexts": [
    {
      "name": "aws-user@kubernetes-on-aws",
      "context": {
        "cluster": "kubernetes-on-aws",
        "user": "aws-user"
      }
    },
    {
      "name": "research",
      "context": {
        "cluster": "test-cluster-1",
        "user": "dev-user"
      }
    },
    {
      "name": "test-user@development",
      "context": {
        "cluster": "development",
        "user": "test-user"
      }
    },
    {
      "name": "test-user@production",
      "context": {
        "cluster": "production",
        "user": "test-user"
      }
    }
  ],
  "current-context": "test-user@development"
}

## Consulta el nombre de la clave users.name y exporta el comando a un fichero
cat userslist.json | jpath $.users[*].name
[
  "aws-user",
  "dev-user",
  "test-user"
]
echo "cat userslist.json | jpath $.users[*].name" > /root/
answer18.sh
