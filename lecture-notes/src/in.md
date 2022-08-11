# Introducción

El curso no solo está orientado a obtener la certificación de kubernetes. Se busca obtener conocimientos solidos en la instalación, configuración y resolución de problemas de kubernetes.

## Prerequisitos y objetivos

Se necesitan los siguiente conocimientos:

- Docker
- Básicos de kubernetes (pod, deployments, services)
- YAML
- Entorno básico de pruebas con Virtual Box

Objetivos:

- Core concepts
    - Cluster arquitecture
    - API primitives
    - Services and other network primitives
- Scheduling
    - Label and selectors
    - Daemon sets
    - Resources limits
    - Multiples schedules
    - Manual sheduling
    - Scheduler events
    - Configure Kubernetes Scheduler
- Loggins monitoring
    - Monitor cluster components
    - Monitor cluster components logs
    - Monitor applications
    - Application logs
- Application Lifecycle Management
    - Rolling updates and rollbacks in deploy
    - Configure applications
    - Scale applications
    - Sel-Healing Application
- Cluster Maintence
    - Cluster upgrade process
    - Operation system upgrades
    - Backup and restore methodologies
- Security
    - Authenthication and authorization
    - Kubernetes security
    - Network policies
    - TLS certificates for cluster components
    - Images securely
    - Security contexts
    - Secure persintants value store
- Storage
    - Persistant volumes
    - Access modes for volumes
    - Persistant volumes claims
    - Kubernetes storage object
    - Configure applications with persistant storage
- Networking Pre-requisites
    - Network, switching, routing, tools
    - DNS and CoreDNS
    - Network namespaces
    - Networking in Docker
- Networking
    - Networking configuration on cluster nodes
    - POD Networking concepts
    - Service Networking
    - Network Loadbalancer
    - Ingress
    - Cluster DNS
    - CNI
- Installation, configurarion and validation
    - Design a kubernetes cluster
    - Install kubernetes master and nodes
    - Secure cluster comunication
    - HA Kubernetes cluster
    - Kubernetes
    - Provision infrastructure
    - Choose a network solution
    - Kubernetes config
    - Run and analyze end-to-end test
    - Node end-to-end tests
- Troubleshooting
    - Application failure
    - Control plane failure
    - Worker node failure
    - Networking

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

- `master` manage, plan, schedule. monitor nodes
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
 
`ETCD` es sistema de almacenamiento clave-valor (key-value store) seguro y distribuido. Es simple, seguro y rápido.

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
./etcdctl set key1 value1
./etcdctl get key1 
```

## ETCD en kubernetes

### Contexto

ETCD es la base de datos de kubernetes, es una key-value store y se utiliza para almacenar información relativa a nodes, pods, configs, secrets, accounts, roles, bindings y otros.

Solo se considera que se ha realizado un cambio en el cluster de kubernetes cuando este se ha actualizad en el ETCD server.

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

y configurando ETCD como service en el nodo master

```bash

```

Investigar cual es el repo más adecuado para obtener el archivo `.tar.gz`. Algunas referencias;

- [https://github.com/etcd-io/etcd/](https://github.com/etcd-io/etcd/)
- [https://etcd.io/docs/v3.5/op-guide/](https://etcd.io/docs/v3.5/op-guide/)



