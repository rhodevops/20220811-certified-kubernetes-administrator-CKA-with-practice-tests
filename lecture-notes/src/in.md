# Introducción

El curso no solo está orientado a obtener la certificación de kubernetes. Se busca obtener conocimientos solidos en la instalación, configuración y resolución de problemas de kubernetes.

## Prerequisitos y objetivos

Se necesitan los siguientes conocimientos:

- Docker
- Básicos de kubernetes (pod, deployments, services)
- YAML
- Entorno básico de pruebas con Virtual Box

## Certificación

El `CKA exam` es un examen basado en laboratorios.

Algunas referencias útiles:

[Certified Kubernetes Administrator](https://www.cncf.io/certification/cka/)
[Exam Curriculum (Topics)](https://github.com/cncf/curriculum)
[Candidate Handbook](https://www.cncf.io/certification/candidate-handbook)
[Exam Tips](http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD)

## Notas de referencia para las sesiones y los laboratorios

[A repository with notes, links to documentation and answers to practice questions here](https://github.com/kodekloudhub/certified-kubernetes-administrator-course)

# Conceptos esenciales

## Arquitectura de un cluster

Un cluster de kubernetes es un conjunto de máquinas físicas o virtuales llamadas nodos que realizan tareas. 

Hay una primera distinción entre los nodos de un cluster. Estos pueden ser:

- `master` manage, plan, schedule, monitor nodes
- `worker nodes` host applications as containers

Componentes del `master`:

- `etcd` database que almacena información en el formato clave-valor
- `kube-scheduler` identifica el nodo apropiado para alojar un contenedor
- `kube-controller-manager` 
    - `node-controller` controla el números de nodos
    - `replication-controller` controla el números de replicas
- `kube-apiserver` controla toda la operativa y orquestación del cluster, expone fuera el API de kubernetes
- `container-runtime-engine` permite utilizar la tecnología container

Componentes de un nodo `worker`:

- `container-runtime-engine` permite utilizar la tecnología container
- `kubelet` es el agente que se ejecuta en cada nodo, escucha las instrucciones del `kube-apiserver` y despliega o destruye contenedores en el nodo
- `kube-proxy` permite la comunicación internodal e intranodal entre contenedores

## ETCD

Vamos a ver aspectos fundamentales del `etcd`. Más adelante en el curso, se hablará de sistemas distribuídos, como opera el etcd, el protocolo RAFT y las mejoras prácticas en relación al número de nodos.

### ¿Qué es el ETCD?
 
`ETCD` es un sistema de almacenamiento clave-valor (key-value store) seguro y distribuido. Es simple, seguro y rápido.

Es importante distinguir entre una una `tabular/relational database`

| Name      | Age | Location |
| --------- | --- | -------- |
| J. Doe    | 45  | Boston   |
| F. Alonso | 23  | Madrid   |

y una `key-value store`

| Key      | Value  | 
| -------- | ------ | 
| Name     | J. Doe | 
| Age      | 45     |
| Location | Boston |


### Instalar ETCD

Pasos:

- Descargar los binarios `curl -L {}`
- Extraer `tar xzvf {}`
- Ejecutar el ETCD Service `./etcd`

Por defecto, el servicio escucha por el puerto `2379`. Se puede adjuntar cualquier cliente al servicio para que almacene y extraiga información.

El cliente por defecto es el cliente de CLI `etcdctl`

Para añadir una entrada en la base de datos y para extraer un valor se utilizan las siguientes órdenes:

```bash
etcdctl set key1 value1
etcdctl get key1 
```

## ETCD en kubernetes

### Contexto

ETCD es la base de datos de kubernetes, es una key-value store y se utiliza para almacenar información relativa a nodes, pods, configs, secrets, accounts, roles, bindings y otros.

Solo se considera que se ha realizado un cambio en el cluster de kubernetes cuando este se ha actualizado en el ETCD server.

### Despliegue de la ETCD

Dependiendo del setup del cluster, el despliegue del ETCD se hace de una manera u otra. Veremos dos casos:

- Cluster configurado desde cero (from scratch)
- Cluster configurado con `kubeadmin`

#### Setup - manual

Si se ha configurado el cluster desde cero, hay que desplegar el ETCD descargando e instalando los binarios 

```bash
wget -q --http-only \
    "httpds://github.com/[...].tar.gz"
```

y configurando ETCD como service en el nodo master. Hay muchos opciones que pueden pasarse al servicio. Algunas de ellas son relativas a certificados. Más adelante en el curso, se verán los certificados TLS:

```bash
# etcd.service
ExecStart=/usr/local/bin/etcd \\
    --name ${ETCD_NAME}
    --cert-file=/etc/etcd/kubernetes.pem \\
    [...]
    --advertise-client-urls https://${INTERNAL_IP}:2379 \\
    [...]
```

La opción `--advertise-client-urls` 

- Se utiliza para especificar la dirección (ip del servidor y puerto) por la que escucha el ETCD.
- El puerto por defecto es `2379`
- Es la url que debería configurarse en el `kube-api-server` cuando este intenta alcanzar el servicio ETCD.

Investigar cual es el repo más adecuado para obtener el archivo `.tar.gz`. Algunas referencias;

- [https://github.com/etcd-io/etcd/](https://github.com/etcd-io/etcd/)
- [https://etcd.io/docs/v3.5/op-guide/](https://etcd.io/docs/v3.5/op-guide/)

#### Setup - kubeadmin

Si se ha configurado el cluster con kubeadmin, entonces kubeadmin despliega el servicio ETCD como pod en el namespace `kube-system`. 

```bash
kubectl get pods exec -n kube-system
```

Dentro de este pod, `etcd-master`, se puede utilizar el cliente `etcdctl` para explorar la base de datos etcd. Por ejmplo, podemos listar todas las claves almacenadas por kubernetes:

```bash
kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

### ETCD en un HA Environment

En un entorno de alta disponibilidad, se tienen varios nodos master y por lo tanto se tendrán múltiples instancias ETCD. En este caso, hay que utilizar un parámetro específico en la configuración:

```bash
# etcd.service
ExecStart=/usr/local/bin/etcd \\
    [...]
    --initial-cluster controller-0=https://{CONTROLLER0_IP}:2380,controller-10=https://{CONTROLLER1_IP}:2380 \\
    [...]
```

### Comandos de ETCD

El cliente `etcdctl` puede interactuar con el servidor ETCD usando dos versiones de la API, la 2 y la 3. Por defecto se usa la 2.

Algunos comandos de la versión 2:

```bash
etcdtl backup
etcdctl cluster-health
etcdctl mk
etcdctl mkdir
etcdctl set
```

Estos comandos cambian en la versión 3:

```bash
etcdctl snapshot save 
etcdctl endpoint health
etcdctl get
etcdctl put
```

Para establecer la versión se puede utiliza la siguiente variable de entorno

```bash
export ETCDCTL_API=3
```

Aparte, hay que especificar la ruta de los archivos de certificados para que el cliente `etcdctl` pueda autenticarse con el `ETCD API Server`. Estos están disponibles en las siguiebtes rutas dentro del contenedor `etcd-master`:


```bash
--cacert /etc/kubernetes/pki/etcd/ca.crt     
--cert /etc/kubernetes/pki/etcd/server.crt     
--key /etc/kubernetes/pki/etcd/server.key
```

Se utilizará el siguiente comando:

```bash
kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 
```

## Kube-api server

El `Kube-apiserver` es el componente del control plane que expone la API de kubernetes. Se encarga de servir las operaciones REST