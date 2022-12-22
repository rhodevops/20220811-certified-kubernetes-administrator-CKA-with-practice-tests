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
 
`etcd` es un sistema de almacenamiento clave-valor (key-value store) seguro y distribuido. Es simple, seguro y rápido.

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
> etcdctl set key1 value1
> etcdctl get key1 
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
> wget -q --http-only \
    "httpds://github.com/[...].tar.gz"
```

y configurando ETCD como service en el nodo master. 

Hay muchos parámetros que pueden pasarse al servicio. Algunos de ellos son relativos a certificados. Más adelante en el curso, se verán los certificados TLS:

```bash
# etcd.service
ExecStart=/usr/local/bin/etcd \\ # faltan parámetros
    --name ${ETCD_NAME}
    --cert-file=/etc/etcd/kubernetes.pem \\
    --advertise-client-urls https://${INTERNAL_IP}:2379 \\
```

El parámetro `--advertise-client-urls` 

- Se utiliza para especificar la dirección (ip del servidor y puerto) por la que escucha el ETCD.
- El puerto por defecto es `2379`
- Es la url que debería configurarse en el `kube-api-server` cuando este intenta alcanzar el servicio ETCD.

Investigar cual es el repo más adecuado para obtener el archivo `.tar.gz`. Algunas referencias;

- [https://github.com/etcd-io/etcd/](https://github.com/etcd-io/etcd/)
- [https://etcd.io/docs/v3.5/op-guide/](https://etcd.io/docs/v3.5/op-guide/)

#### Setup - kubeadmin

Si se ha configurado el cluster con kubeadmin, entonces kubeadmin despliega el servicio ETCD como pod en el namespace `kube-system`. 

```bash
> kubectl get pods exec -n kube-system
```

Dentro de este pod, `etcd-master`, se puede utilizar el cliente `etcdctl` para explorar la base de datos etcd. Por ejmplo, podemos listar todas las claves almacenadas por kubernetes:

```bash
> kubectl exec etcd-master -n kube-system etcdctl get / --prefix -keys-only
```

### ETCD en un HA Environment

En un entorno de alta disponibilidad, se tienen varios nodos master y por lo tanto se tendrán múltiples instancias ETCD. En este caso, hay que utilizar un parámetro específico en la configuración:

```bash
# etcd.service
ExecStart=/usr/local/bin/etcd \\ 
    --initial-cluster controller-0=https://{CONTROLLER0_IP}:2380,controller-10=https://{CONTROLLER1_IP}:2380 \\
```

### Comandos de ETCD

El cliente `etcdctl` puede interactuar con el servidor ETCD usando dos versiones de la API, la 2 y la 3. Por defecto se usa la 2.

Algunos comandos de la versión 2:

```bash
> etcdtl backup
> etcdctl cluster-health
> etcdctl mk
> etcdctl mkdir
> etcdctl set
```

Estos comandos cambian en la versión 3:

```bash
> etcdctl snapshot save 
> etcdctl endpoint health
> etcdctl get
> etcdctl put
```

Para establecer la versión se puede utiliza la siguiente variable de entorno

```bash
> export ETCDCTL_API=3
```

Aparte, hay que especificar la ruta de los archivos de certificados para que el cliente `etcdctl` pueda autenticarse con el `ETCD API Server`. Estos están disponibles en las siguiebtes rutas dentro del contenedor `etcd-master`:


```bash
/etc/kubernetes/pki/etcd/ca.crt     
/etc/kubernetes/pki/etcd/server.crt     
/etc/kubernetes/pki/etcd/server.key
```

Se utilizará el siguiente comando:

```bash
> kubectl exec etcd-master -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt  --key /etc/kubernetes/pki/etcd/server.key" 
```

## Kube API server

El `Kube-apiserver` es el componente del control plane que expone la API de kubernetes. Se encarga de manejar las peticiones de tipo REST. 

Cuando se ejecuta un comando `kubectl`, se hace una llamada a al `kube-api server`, que autentica y valida la petición. Entonces recibe la información de `etcd` y respionde de vuelta con la información pedida. Realmente no sería necesario utilizar la herramienta kubectl, sino que se podría invocar directamente la API.

```bash
> curl -X POST /api/v1/namespaces/default/pods ...[other]
```

Proceso:

1. Authenticate User
2. Validate Request
3. Retrieve data
4. Update ETCD
5. Scheduler
6. Kubelet

El `Kube-apiserver` es el único componente que interactúa directamente con la base de datos `etcd`. Se puede describir como el centro que gestiona las diferentes tareas necesarias para realizar un cambio en el cluster.

Si se configura kubernetes mediante la "hard way", hay que saber que el `Kube-apiserver` está disponible como binarios en la página de kubernetes. Se deberá descargar y configurar para que se ejecute como servicio en el nodo master. Habrá que configurar varias parámetros y al igual que con otros componentes, son de especial importancias los parámetros relativos a los certificados. 

Es muy importante tener claro que todos los componentes del cluster están interrelacionados y que cada componente debe conocer dónde se encuentra el resto.

Si has configurado el cluster de kubernetes con kubeadmin, entonces el `kube-apiserver` se ha desplegado como pod en el namespace `kube-system` en el nodo máster.

```bash
> kubectl get pods -n kube-system
```

Para consultar el manifiesto utilizado en la definición del pod y en la definición del service se puede ordenar lo siguiente:

```bash
> cat /etc/kubernetes/manifests/kube-apiserver.yaml
> cat /etc/kubernetes/manifests/kube-apiserver.service
```

Para ver el proceso del kube-apiserver y las opciones de parámetro efectivas, se puede ejecutar en el nodo master:

```bash
> ps -aux | grep kube-apiserver
```

## Kube Controller Manager

El `kube-controller-manager` gestiona varios controladores dentro de kubernetes. Es responsable de monitorizar y llevar a cabo acciones de repuesta. 

Proceso:

1. Watch Status
2. Remediate Situation

Hay dos tipos de controladores que son muy importante:

- `Node-Controller` Controla el estado de los nodos para que no se caiga la aplicación que se ejecuta en ellos.
    - Node Monitor Period = 5s
    - Node Monitor Grace Period = 40s
    - POD Eviction Timeout = 5m
- `Replication-Controller` Controla el número de replicas y se asegura de que estén levantados el número de pods deseado. Si el pod se cae, levanta otro.

Sobre los parámetros mencionados:

- `Node Monitor Period` Periodo de monitorización.
- `Node Monitor Grace Period` Periodo de gracia desde el momento en que el nodo se cae hasta que se marca como `unreachable`
- `POD Eviction Timeout` Tiempo de espera para que el nodo vuelva a levantar. Si no lo hace, los pods se realojan en un nodo sano si estos forman parte de un Replica-Set.

Aparte de los mencionadosn, hay otros tipos de controladores:

- `Deployment-Controller`
- `Namespace-Controller`
- `Job-Controller`
- `CronJob-Controller`
- `Endpoint-Controller`
- etc

Cuando se instala el `kube-controller-manager`, se instalan también los diferentes tipos de controladores.

Al igual que con otros componentes, para intalarlo se hace lo habitual:

- Descargar/Extraer el binario `kube-controller-manager`
- Ejecutarlo como servicio

Se muestra a continuación algun parámetro de la configuración:

```bash
# kube-controller-manager.service
ExecStart=/usr/local/bin/kube-controller-manager \\ 
    --addres=0.0.0.0 \\
    --cluster-cidr=10.200.0.0/16 \\
    --node-monitor-period=5s \\
    --node-monitor-grace-period=40s \\
    --pod-eviction-timeout=5m0s \\
```

Si has configurado el cluster de kubernetes con kubeadmin, entonces el `kube-controller-manager` se ha desplegado como pod en el namespace `kube-system` en el nodo máster.

```bash
> kubectl get pods -n kube-system
```

Para consultar el manifiesto utilizado en la definición del pod y en la definición del service se puede ordenar lo siguiente:

```bash
> cat /etc/kubernetes/manifests/kube-controller-manager.yaml
> cat /etc/kubernetes/manifests/kube-controller-manager.service
```

Para ver el proceso del kube-controller-manager y las opciones de parámetro efectivas, se puede ejecutar en el nodo master:

```bash
> ps -aux | grep kube-controller-manager
```

## Kube Scheduler

`kube scheduler` es el responsable de la programación de los nodos y los pods. Es decir, se encarga de decidir que pod se aloja en que nodo. Pero no se encarga de crearlo, eso lo hace el agente `kubelet` del nodo.

Normalmente, se tendrán pods con distintos requerimientos. Proceso de alojo de un pod en un nodo:

1. **Descartar nodos** Se descartan los nodos que no se adaptan al pérfil del pod (p.e. por insuficiente CPU)
2. **Clasificar nodos** En función de varios parámetros (p.e. cantidad de CPU que queda libre tras la asignación)

Al igual que con otros componentes, para intalarlo se hace lo habitual:

- Descargar/Extraer el binario `kube-scheduler`
- Ejecutarlo como servicio

Se muestra a continuación algun parámetro de la configuración:

```bash
# kube-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler \\ 
    --config=/etc/kubernetes/config/kube-scheduler.yaml \\
    --v=2 
```

Si has configurado el cluster de kubernetes con kubeadmin, entonces el `kube-scheduler` se ha desplegado como pod en el namespace `kube-system` en el nodo máster.

```bash
> kubectl get pods -n kube-system
```

Para consultar el manifiesto utilizado en la definición del pod se puede ordenar lo siguiente:

```bash
> cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

Para ver el proceso del kube-scheduler y las opciones de parámetro efectivas, se puede  ejcutar en el nodo master:

```bash
> ps -aux | grep kube-scheduler
```

## Kubelet

`kubelet` es el agente que se ejecuta en el nodo. 

Algunas funciones:

- Crear los pods en el nodo siguiendo las instrucciones del scheduler.
- El kubelet envía la petición al container runtime (docker).
- Monitorizar el nodo y los pods.

***Importante: kubeadmin no despliega kubelets** A diferencia que con los componentes descritos hasta ahora, la opción kudeadmin no despliega los kubelets automáticamente en forma de pods, sino que hay que instalarlos de forma manual en los nodos worker.

El proceso es el habitual:

- Descargar/Extraer el binario `kubelet`
- Ejecutarlo como servicio

Se muestra a continuación algun parámetro de la configuración:

```bash
# kube-scheduler.service
ExecStart=/usr/local/bin/kubelet \\ # faltán parámetros
    --config=/var/lib/kubelet/kube-config.yaml \\
    --container-runtime=remote \\
```

Para ver el proceso del kubelet y las opciones de parámetro efectivas, se puede  ejecutar en el nodo worker:

```bash
> ps -aux | grep kubelet
```

## Kube Proxy

### Contexto y necesidad

En un cluster de kuberentes cualquier pod tiene que poder alcanzar cualquier otro pod. Esto se satisface desplegando una solución de POD networking. 

- Una POD network es una red virtual interna que se expande a través de los nodos del clúster conéctándolos.
- Hay varias solucionens disponibles.

Imaginemos que tenemos el siguiente cluster de dos nodos:

```latex
 +-----------------------------------------------+
 | Cluster de dos nodos                          |
 +-----------------------------------------------+
---------------------------------------------------
 POD Network
 +-----------------------+-----------------------+
 | Nodo 1                | Nodo 2                |
 | +-------------------+ | +-------------------+ |
 | | Pod: webapp       | | | Pod: database     | |
 | | 10.32.0.14        | | | 10.32.0.15        | |
 | +-------------------+ | +-------------------+ | 
 | kube-proxy            | kube-proxy            |
 +-----------+-----------+-----------+-----------+
             |                       |
             +----- service: db -----+
                     10.96.0.12
```

Varias cosas que hay que saber:

- La webapp se podría alcanzar la database utilizando la IP pero no es lo recomendable por todas las razones ya conocidad.
- El service `db` se utiliza para exponer la database y pueda llegar hasta ella la webapp.
- El service es accesible desde cualquier nodo del clúster.

La pregunta es: ¿El service `db` está unido a la POD Network? La respuesta es NO. El service no es un contenedor y por tanto no tiene una interface o un proceso escuchando de forma activa.

El service es un componente virtual que **solamente vive en la memoria de kubernetes**. ¿Cómo se alcanca entonces el service? Aqui es donde enta el kube-proxy.

El `kube-proxy` es un proceso que se ejecuta en cada nodo del clúster de kubernetes. Su cometido es buscar nuevos servicios. 

Cada vez que se crea un nuevo servicio, el kube-proxy crea las reglas apropiedadas en cada nodo para reenviar el tráfico a esos servicios. Una forma de hacerlo es utilizando reglas `iptables`.

**Información** Iptables es una herramienta linux que se encarga de analizar cada uno de los paquetes del tráfico de red que entra en una máquina y decidir, en función de un conjunto de reglas, qué hacer con ese paquete.

En este caso, kube-proxy crea una regla de tablas ip en cada nodo del clúster para reenviar el tráfico que se dirije a la IP del servicio, que es `10.96.0.12`, a la IP del nodo actual que es `10.32.0.15`

```latex
 +-----------------------------------------------+
 | Cluster de dos nodos                          |
 +-----------------------------------------------+
---------------------------------------------------
 POD Network
 +-----------------------+-----------------------+
 | Nodo 1                | Nodo 2                |
 | +-------------------+ | +-------------------+ |
 | | Pod: webapp       | | | Pod: database     | |
 | | 10.32.0.14        | | | 10.32.0.15        | |
 | +-------------------+ | +-------------------+ | 
 | kube-proxy            | kube-proxy            |
 | IP table rule:        | IP table rule:        |
 | 10.96.0.12            | 10.96.0.12            |
 | -> 10.32.0.15         | -> 10.32.0.15         |
 +-----------+-----------+-----------+-----------+
             |                       |
             +----- service: db -----+
                     10.96.0.12
```

### Instalación de Kube Proxy

Al igual que con otros componentes, para intalarlo se hace lo habitual:

- Descargar/Extraer el binario `kube-proxy`
- Ejecutarlo como servicio

Se muestra a continuación algun parámetro de la configuración:

```bash
# kube-scheduler.service
ExecStart=/usr/local/bin/kube-scheduler \\ 
    --config=/var/lib/kube-proxy/kube-proxy-config.yaml 
Restart=on-failure
RestartSec=5
```

Si has configurado el cluster de kubernetes con kubeadmin, entonces el `kube-proxy` se ha desplegado como daemon-set en el namespace `kube-system`, de modo que hay un pod desplegado en cada nodo: 

```bash
> kubectl get daemonset -n kube-system
```

## Recap. Sobre los pods de kubernetes

Sea el siguiente escenario:

- Tenemos una aplicación desarrollada y construída como imagen de docker.
- La imagen se encuentra en un repo al que puede acceder kubernetes.
- Tenemos levantado un clúster de kubernetes que se encuentra operativo.

Se recuerda que el cometido principal de kubernetes es deplegar nuestra aplicación en forma de contenedores que se ejecutan en una serie de máquinas configuradas como nodos worker dentro del clúster. 

Sin embargo, kubernetes no despliega los contenedores directamente en los nodos sino que los encapsula como pods. Un pod es la instancia unitaria de la aplicación y el objeto mínimo que se puede crear en kubernetes.

- En un mismo pod, no existen contenedores de un mismo tipo.
- La relación 1/1 entre pods y contenedores es muy habitual.
- Un pod multi-container contiene otros contenedores que dan soporte al contenedor principal. Estos contenedores nacen y mueren juntos.

Para escalar la aplicación se hace lo siguiente:

- Desplegar nuevas intancias de la aplicación en forma de pods en el mismo nodo.
- Crear un nuevo nodo para desplegar más pods que contienen nuevas instancias de la aplicación.

Consultar los apuntes del curso `20220711-kubernetes-for-the-absolute-beginners`. Ver la sección **Conceptos de kubernetes** para refescar lo siguiente:

- Los conceptos básicos acerca de la necesidad de los pods.
- La utilidad de terminal `kubelet`

## Recap. Pods with YAML

Consultar la sección correspondiente dentro de **[Conceptos de kubernetes: Pods, ReplicaSets, Deployments]** en los apuntes siguientes:

> `20220711-kubernetes-for-the-absolute-beginners`.

## Lab. Pods

Algunos comandos utilizados:

```bash
kubectl get pods
kubectl apply -f <name-pod.yaml>
kubectl run nginx --image=nginx
kubectl describe pods
kubectl get nodes
kubectl describe pods <pod_name> | grep "Container ID"
kubectl delete pod <pod_name>
```
    
## Recap. Replica Sets

Consultar la sección correspondiente dentro de **[Conceptos de kubernetes: Pods, ReplicaSets, Deployments]** en los apuntes siguientes:

> `20220711-kubernetes-for-the-absolute-beginners`.

## Lab. Replica Sets

Algunos comandos utilizados:

```bash
kubectl get rs
kubectl get rs -o wide
kubectl edit rs <replica-set-name>
kubectl scale --replicas=5 rs <replica-set-name>
```

## Recap. Deployments

Consultar la sección correspondiente dentro de **[Conceptos de kubernetes: Pods, ReplicaSets, Deployments]** en los apuntes siguientes:

> `20220711-kubernetes-for-the-absolute-beginners`.

## Lab. Deployments

Algunos comandos utilizados:

```bash
kubectl get pods
kubectl get rs
kubectl get deploy
kubectl get deploy -o wide
kubectl apply -f deployment-definition-1.yaml
kubectl create deployment --image=httpd:2.4-alpine httpd-fronted --replicas=3 --dry-run=client -o yaml > mydeployment.yaml
```

## Recap. Services

Consultar el capítulo **[Services]** en los apuntes siguientes:

> `20220711-kubernetes-for-the-absolute-beginners`.

## Lab. Services

Algunos comandos utilizados:

```bash
kubectl get svc
kubectl get svc -o wide
kubectl describe svc
```

## Namespaces

### Namespaces en kubernetes

Un `namespace` es un espacio aislado de nombres, es decir, un conjuntos de nombres utilizados para identificar y hacer referencia a un conjunto de objetos. El namespace asegura que todos los objetos que aloja tienen un nombre único. 

Cuando se configura un clúster de kubernetes, se crean automáticamente los siguientes namespaces:

- `default`
- `kube-system` contiene los pods y servicios para propósitos internos requeridos para la solución de networking.
- `kube-public` alojará los recursos que deben ser accesibles para todos los usuarios.

En el ámbito empresarial, los clusters típicamente tienen bastante namespaces.

Hay muchas posibles soluciones para aislar los entornos de DEV y de PRO (clusteres diferentes usando contextos). Una de ellas es utilizar un namespace llamo `Dev` y otro namespace llamado `Pro`. 

Cada uno de estos namespaces tiene:

- Sus propias políticas que definan los distintos permisos. 
- Distintas asignaciones respecto a los límites y usos de recursos.

### Namespaces - DNS

Tenemos un cluster con los siguientes namespaces:

```latex
 +--------------------------------------------------+
 | Cluster                                          |
 +--------------------------------------------------+
------------------------------------------------------
 +----------------+----------------+
 | NS - default   | NS - dev       |
 | +------------+ | +------------+ |
 | | web-pod    | | | web-pod    | |
 | +------------+ | +------------+ |
 | +------------+ | +------------+ |
 | | db-service | | | db-service | |
 | +------------+ | +------------+ |
 | +------------+ | +------------+ |
 | | web-deploy | | | web-deploy | |
 | +------------+ | +------------+ |
 +----------------+----------------+
```

En este caso, la web-app del ns Default puede alcanzar el db-service de su namespace simplemente utilizando el hostname `db-service`

```bash
mysql.connect("db-service")
```

Sin embargo, si quiere alcanzar el db-service del ns Dev, debe utilizar el nombre siguiente

```bash
mysql.connect("db-service.dev.svc.cluster.local")
```

Esto es así porque, cuando el servicio es creado, automáticamente se añade una entrada DNS con este formato:

- `cluster.local` es el nombre de `dominio` por defecto del cluster de kubernetes.
- `svc` es el `subdominio` para el servicio
- `dev` es el `namespace`
- `db-service` es el `nombre` del servicio

## Algunos comandos relacionados con el namespace

Listar pods:

```bash
kubectl get pods #lista en el ns default
kubectl get pods -n kube-system
```

Crear pods:

```bash
kubectl create -f pod-definition.yaml #se aplica al ns default
kubectl create -f pod-definition.yaml -n dev
```

o alternativamente incluir en el manifiesto el siguiente metadata

```yaml
metadata:
    namespace: dev
```

Crear un nuevo namepace. Se puede hace con el siguiente manifiesto

```yaml
#namespace-pro.yaml
apiVersion: v1
kind: Namespace
metadata:
    name: pro
spec:
```

y ejecutando

```bash
kubectl create -f namespace-pro.yaml
```

O alternativamente, directamente se puede ejecutar:

```bash
kubectl create namespace pro
```

Hacer switch de un namespace a otro:

```bash
kubectl config set-context $(kubectl config current-context) --namespace=dev
```

donde `$(kubectl config current-context)` nos devuelve el contexto actual, concepto que todavía no hemos tocado y que se verá más adelante. El contexto viene a ser el cluster actual con el que estamos interaccionando dentro de un contexto de múltiples clústers.

Para listar los pods de todos los namespaces, se ejecuta:

```bash
kubectl get pods --all-namespaces
```

### Resource Quota

Podemos utilizar el objeto `ResourceQuota` para establecer, por ejemplo, los límites de memoria y CPU en un determinado namespace.

Utilzaremos un manifiesto como el siguiente:

```yaml
#compute-quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
    name: compute-quota
    namespace: dev
spec:
    hard:
        pods: "10"
        requests.cpu: "4"
        requests.memory: 5Gi
        limits.cpu: "10"
        limits.memory: 10Gi
```

y lo crearemos ejecutando

```bash
kubectk create -f compute-quota.yaml
```

## Lab. Namespaces

Algunos comandos utilizados:

```bash
kubectl get ns --no-headers | wc -l
kubectl get pods -n research --no-headers | wc -l
kubectl run redis --image=redis -n finance
kubectl get pods --all-namespaces
kubectl get svc --all-namespaces
```

## Imperative vs Declarative

### Aproximación imperativa y declarativa en kubernetes

En el mundo de la IaaC o Infraestructura como código, existen dos aproximaciones distintas para el mantenimiento y gestión de la infraestructura:

- `Forma imperativa` Se especifican las instrucciones que hay que seguir.
- `Forma declarativa` Se especifica el resultado final deseado.

**Forma imperativa**

1. Aprovisionar una vm con el nombre de "web-server".
2. Instalar el software nginx en la máquina.
3. Editar el archivo de configuración para usar el puerto "8080".
4. Editar el archivo de configuración a la ruta web "/var/www/nginx"
5. Cargar las páginas web a "/var/www/nginx" con GIT Repo - X.
6. Iniciar el servidor nginx.

**Forma declarativa**

```yaml
VM Name: werb-server
Package: nginx
Path: /var/www/nginx
Code: GIT Repo - X
```

Existen muchas herramientas devops conocidas como Ansible, Puppet, Chef o Terraform que utilizan la aproximación declarativa:

En kubernetes utilizamos una **aproximación imperativa** cuando ejecutamos comandos como los siguientes

```bash
kubectl run --image=nginx nginx
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80 
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5 
kubectl set image deployment nginx nginx=nginx:1.18
kubectl create -f nginx.yaml
kubectl replace -f nginx.yaml
kubectl delete -f nginx.yaml
```

en ocasiones, sobre un manifiesto `nginx.yaml`.

La **aproximación imperativa** de kubernetes se pone en práctica cuando creamos un manifiesto `nginx.yaml` y utilizamos el comando

```bash
kubectl apply -f nginx.yaml
```

que servirá para aplicar el estado deseado en el manifiesto. En esta aproximación se utiliza el comando `apply` tanto para crear como para actualizar o borrar un objeto.

### Comandos imperativos

Para crear un objeto:

```bash
kubectl run --image=nginx nginx
kubectl create deployment --image=nginx nginx
kubectl expose deployment nginx --port 80 
```

Para actualizar un objeto:

```bash
kubectl edit deployment nginx
kubectl scale deployment nginx --replicas=5 
kubectl set image deployment nginx nginx=nginx:1.18
```

Los comandos anteriores solo se utilizan una vez y tras ejecutarlos no olvidamos de ellos. No es la mejor forma de trabajar en un entorno empresarial, pero puede ser utilir para superar un examen de certificación.

### Archivos de configuración de objetos imperativos

La primera ventaja en el uso de archivos de configuración es que estos archivos pueden almacenarse en un repositorio y podemos tener un control de versiones sobre ellos:

```yaml
#nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels: 
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

Para crear un objeto:

```bash
kubectl create -f nginx.yaml
```

Para actualizar un objeto:

**Opción 1. No tan recomendada** Una opción es ejecutar

```bash
kubectl edit deployment nginx
```

para que se abra un archivo de definición yaml muy similar al que `nginx.yaml` que hemos utilizado para crear los pods pero con algún campo adicional apra almacenar el estado:

```yaml
#pod-definition
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels: 
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx:1.18
status:
  conditions:
  - lastProbeTime: null
    status: "True"
    type: Initialized
```

Los cambios que hagamos sobre este archivo, se aplicarán sobre los pods que se están ejecutando. Sin embargo, observar en el ejemplo la diferencia existente entre los dos archivos de configuración. En el manifiesto aparece la imagen `nginx` y en el archivo de estado aparece la imagen `nginx:1.18`. Esto es porque reaamente, **el comando edit no se graba en ningún sitio después de que el cambio es efectuado**. 

Si en el futuro, alguien quiere modificar el archivo local del manifiesto yaml, no sabrá que otra persona (o ella misma) realizó en el pasado un cambio utilizando el comando kubectl.

**Opción 2. Recomendada** La mejor opción es modificar el manifiesto yaml original para cambiar, por ejemplo, la imagen a `nginx:1.18` y ejecutar el comando

```bash
kubectl replace -f nginx.yaml
```

y de esta forma se puede hacer un control de las distintas versiones.

En el caso de que se quiera una destrucción y recreación completa del objeto se utilizará el flag `--force`, es decir, se ejecutará el comando

```bash
kubectl replace --force -f nginx.yaml
```

Es importante saber, que si el objeto ya esta creado y se utiliza el comando `create`
saltará un error. Y del mismo modo, se producirá un error si el objeto no está creado y se intenta ejecutar un `replace`. 

Esto último es una desventaja frente a la aproximación declarativa que estamos a punto de tratar.

### Aproximación declarativa en kubernetes

En esta aproximación se trabaja siempre con el mismo manifiesto `nginx.yaml`

```yaml
#nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
  labels: 
    app: myapp
    type: front-end
spec:
  containers:
    - name: nginx-container
      image: nginx
```

Para crear un objeto se utiliza el comando inteligente

```bash
kubectl apply -f nginx.yaml
```

que creará un objeto en el caso de que no exista. En el caso de que existan **varios archivos de configuración de objetos** en la ruta `/path/to/config-files`, algo habitual, se puede utilizar el comando

```bash
kubectl apply -f /path/to/config-files
```
para crear a la vez todos los objetos.

Para actualizar un objeto, simplemente actualizamos su manifiesto y ejecutamos el mismo comando de nuevo

```bash
kubectl apply -f nginx.yaml
```

## Tips. Comandos imperativos con kubectl

En los siguient comandos kubectl, son útiles las siguientes dos opciones:

- `--dry-run=client` Utilizando esta opción, el recurso no será creado. Sirve para saber si se puede crear el recurso.
- `-o yaml` Está opción permite imprimir en la pantalla la definición del recurso en formato yaml.

Las opciones anteriores se pueden combinar para generar rápidamente una plantilla para un manifiesto yaml.

### Pods

Create an NGINX Pod

```bash
kubectl run nginx --image=nginx
```

Generate POD Manifest YAML file . Don't create it

```bash
kubectl run nginx --image=nginx --dry-run=client -o yaml > nginx-pod.yaml
```

### Deployments

Create a deployment

```bash
kubectl create deployment --image=nginx nginx
```

Generate Deployment YAML file. Don't create it

```bash
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml
```

Generate Deployment with 4 Replicas

```bash
kubectl create deployment nginx --image=nginx --replicas=4
```

You can also scale a deployment using the kubectl scale command.

```bash
kubectl scale deployment nginx --replicas=4
```

Another way to do this is to save the YAML definition to a file and modify

### Services

Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

```bash
kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors)

Or

```bash
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml
```

(This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:

```bash
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
```

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

```bash
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the kubectl expose command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

References:

- [https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
- [https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)

## Lab. Imperative commands

Algunos comandos utilizados:

```bash
kubectl run nginx-pod --image=nginx:alpine
kubectl run redis --image=redis:alpine --labels tier=db 
kubectl expose pod redis --port=6379 --name redis-service #svc ClusterIP
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
kubectl run custom-nginx --image=nginx --port=8080
kubectl create namespace dev-ns
kubectl create deployment redis-deploy --image=redis --replicas=2 -n dev-ns
kubectl run httpd --image=httpd:alpine --port=80 --expose #svc ClusterIP
```

## Comando kubectl apply

Recordamos la forma de trabajar declarativamente consiste en:

1. Modificar el manifiesto file.yaml donde se define el objeto de kubernetes. Lo llamaremos también `local file`.
2. Ejecutar el comando `kubectl apply -f file.yaml` para que se lleven a cabo los cambios necesarios.

Internamente esto funciona con tres conjuntos de datos:

- `local file` manifiesto yaml de definición, almacenado localmente.
- `live object configuration` yaml con el estado actual del objeto, almacenado en la memoria de kubernetes.
- `last applied configurarion` json con los últimos cambios aplicados, almacenado en el archivo `live object configuration` dentro del campo `anotations`.

Es muy importante no mezclar las aproximaciones imperativa y declarativa.

# Scheduling

## Scheduling manual

Vamos a ver como funciona el scheduling de kubernetes.

Todos los pods tienen el campo `spec.nodeName`. Lo habitual es que este campo no se configure en el manifiesto y que kubernetes lo añada automáticamente.

El scheduler examina todos los pods en busca de aquellos que no tienen configurada esta propiedad y los asigna un nodo creando un `binding object`.

En el caso de que no exista un scheduler para monitorizar y asignar nodos, el pod se quedaría en el estado de `pending`. 

Sin scheduler, la forma manual más sencilla de asignar un nodo a un pod es indicarlo en su manifiesto. Esto debe hacerse antes de crear el pod

```yaml
#pod-definition
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels: 
    app: mginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports: 
        - containerPort: 8080
  nodeName: node02
```

En el caso de que el pod ya este creado, podemos modificar el nodo que un pod tiene asignado creando un objeto de tipo `Binding`

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: nginx
target:
  apiVersion: v1
  kind: Node
  name: node02
```

y lanzando un POST request a la binding API del pod

```bash
curl --header "Content-Type:application/json"  --request POST --data '{"apiVersion":"v1", "kind":"Binding" ... }' http://$SERVER/api/namespaces/default/pods/binding/
```

Observar que esto requiere convertir el archivo yaml a formato json.

## 

