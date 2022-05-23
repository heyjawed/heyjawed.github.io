---
title:  "Setup Hashicorp Vault cluster on kubernetes"
# image : "/assets/images/post/post-1.jpg"
author: "Jawed Salim"
date: 2022-05-23 11:35:58 +0600
description : "Vault Consul Setup on k8s in HA mode"
tags: [Vault-Consul]
---


The [Vault Helm chart](https://github.com/hashicorp/vault-helm) is the recommended way to install and configure Vault on Kubernetes. In addition to running Vault itself, the Helm chart is the primary method for installing and configuring Vault to integrate with other services such as Consul for High Availability (HA) deployments.

While the Helm chart automatically sets up complex resources and exposes the configuration to meet your requirements, it does not automatically operate Vault. You are still responsible for learning how to _initialize, monitor, backup, upgrade_, etc. the Vault cluster.

**Steps we will use in this guide**

- Create Kubernetes Namespace
- Set Up HashiCorp Helm Repo
- Install Consul Storage backend
- Install Vault in HA mode
- Initialize and Unseal Vault


## Create Kubernetes Namespace

Create a K8s namespace.

```sh
$ kubectl create ns vault-consul

namespace/vault-cluster created
```

View your new K8s objects.

```sh
$ kubectl get all -n vault-consul
```

## Set Up HashiCorp Helm Repo

Helm must be installed and configured on your machine. For help installing Helm, please refer to the [Helm documentation](https://helm.sh/) or the [Vault Installation to Minikube via Helm](https://learn.hashicorp.com/tutorials/vault/kubernetes-minikube) tutorial.

Check Helm Version

```sh
$ helm version

version.BuildInfo{Version:"v3.4.1", GitCommit:"c4e74854886b2efe3321e185578e6db9be0a6e29", GitTreeState:"dirty", GoVersion:"go1.15.4"}
```

To access the Vault and Consul Helm chart, add the Hashicorp Helm repository.

```sh
$ helm repo add hashicorp https://helm.releases.hashicorp.com

"hashicorp" has been added to your repositories
```

Check that you have access to the `Consul` chart.

```sh
$ helm search repo hashicorp/consul

NAME            	CHART VERSION	APP VERSION	    DESCRIPTION
hashicorp/consul	0.43.0       	1.12.0     	    Official HashiCorp Consul Chart
```

Check that you have access to the `Vault` chart.

```sh
$ helm search repo hashicorp/vault

NAME           	    CHART VERSION	APP VERSION	    DESCRIPTION
hashicorp/vault	    0.19.0       	1.9.2      	    Official HashiCorp Vault Chart
```

## Install Consul Storage backend

The Consul storage backend is used to **persist Vault's data in Consul's key-value store**. In addition to providing durable storage, inclusion of this backend will also register Vault as a service in Consul with a default health check.
**High Availability** – the Consul storage backend supports high availability.

Now you're ready to install Consul! Install Consul on the dedicated namespace.

```sh
$ helm install consul hashicorp/consul --set global.name=consul -n vault-consul

NAME: consul
LAST DEPLOYED: Tue Apr 21 17:01:37 2022
NAMESPACE: vault-consul
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Consul!

Your release is named consul.

To learn more about the release, run:

  $ helm status consul
  $ helm get all consul

Consul on Kubernetes Documentation:
https://www.consul.io/docs/platform/k8s

Consul on Kubernetes CLI Reference:
https://www.consul.io/docs/k8s/k8s-cli
```

Check k8s objects in `vault-consul` namespace

```sh
$ kubectl get all -n vault-consul --selector=app=consul

NAME                      READY   STATUS    RESTARTS   AGE
pod/consul-client-7dhmg   1/1     Running   0          2m
pod/consul-client-jmlbd   1/1     Running   0          2m
pod/consul-client-nzwfp   1/1     Running   0          2m
pod/consul-server-0       1/1     Running   0          2m
pod/consul-server-1       1/1     Running   0          2m
pod/consul-server-2       1/1     Running   0          2m

NAME                    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                                                                   AGE
service/consul-dns      ClusterIP   10.8.8.105   <none>        53/TCP,53/UDP                                                             2m
service/consul-server   ClusterIP   None         <none>        8500/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP,8600/UDP   2m
service/consul-ui       ClusterIP   10.8.2.186   <none>        80/TCP                                                                    2m

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/consul-client   3         3         3       3            3           <none>          2m

NAME                             READY   AGE
statefulset.apps/consul-server   3/3     2m
```

> Consul storage backend is installed and Vault can access it using address - `consul-server:8500` (Consul server service name and port) in same namespace

## Install Vault in HA mode

Get `values.yaml` file to update config options

```sh
$ wget https://raw.githubusercontent.com/hashicorp/vault-helm/v0.19.0/values.yaml
```

Now open `values.yaml` in your favourite editor and update following config options

```yaml
# Available parameters and their default values for the Vault chart.

global:
  enabled: true
  # TLS for end-to-end encrypted transport
  tlsDisable: true
...
...
injector:
  # True if you want to enable vault agent injection.
  enabled: true
...
...
server:
  # If not set to true, Vault server will not be installed.
  enabled: true
  ...
  ...
  ha:
    enabled: true
    replicas: 3
    ...
    ...
    config: |
      ui = true

      listener "tcp" {
        tls_disable = 1
        address = "[::]:8200"
        cluster_address = "[::]:8201"
      }
      storage "consul" {
        path = "vault"
        address = "consul-server:8500" # <consul-server-service:8500>
      }

      service_registration "kubernetes" {}
...
...
```

Once you updated the `values.yaml` file, install `vault`

```sh
$ helm install vault hashicorp/vault -n vault-consul -f values.yaml

NAME: vault
LAST DEPLOYED: Tue Apr 21 17:08:31 2022
NAMESPACE: vault-consul
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault
```

Check k8s objects in `vault-consul` namespace

```sh
$ kubectl get all -n vault-consul --selector=app.kubernetes.io/instance=vault

NAME                                       READY   STATUS    RESTARTS   AGE
pod/vault-0                                0/1     Running   0          3m
pod/vault-1                                0/1     Running   0          3m
pod/vault-2                                0/1     Running   0          3m
pod/vault-agent-injector-c6d85cb5f-csfql   1/1     Running   0          3m

NAME                               TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
service/vault                      ClusterIP   10.8.6.32     <none>        8200/TCP,8201/TCP   3m
service/vault-active               ClusterIP   10.8.15.119   <none>        8200/TCP,8201/TCP   3m
service/vault-agent-injector-svc   ClusterIP   10.8.10.120   <none>        443/TCP             30h
service/vault-internal             ClusterIP   None          <none>        8200/TCP,8201/TCP   3m
service/vault-standby              ClusterIP   10.8.10.62    <none>        8200/TCP,8201/TCP   3m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/vault-agent-injector   1/1     1            1           3m

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/vault-agent-injector-c6d85cb5f   1         1         1       3m

NAME                     READY   AGE
statefulset.apps/vault   0/3     3m
```

> Note: Vault is deployed in HA mode with Consul storage backend but the pods are not in ready state, you need to initialize and unseal the Vault


## Initialize and Unseal Vault

After the Vault Helm chart is installed, one of the Vault servers need to be initialized. The initialization generates the credentials necessary to unseal all the Vault servers.

View all the Vault pods in the current namespace:

```sh
$ ❯ kubectl get pods --selector='app.kubernetes.io/name=vault' -n vault-consul

NAME                                        READY   STATUS    RESTARTS   AGE
pod/vault-0                                 0/1     Running   0          4m51s
pod/vault-1                                 0/1     Running   0          4m51s
pod/vault-2                                 0/1     Running   0          4m51s
```

Initialize one Vault server with the default number of key shares and default key threshold:

```sh
$ kubectl exec --stdin=true --tty=true vault-0 -n vault-consul -- vault operator init

Unseal Key 1: MQxywN4svq/cBFxVOemsTbasIA0Z8qwnFyOI3p3SFDv3
Unseal Key 2: QGAHFZ+q6tXR6tCSQ7cekWTURzda3mxohquXv7va+6AX
Unseal Key 3: vHuHc3ZNCI9mZ8x+TyAbXmjavdJTGXDCLD7aSJfHjx6T
Unseal Key 4: 6hHPhx8hQsjQgeCj8BspL+gPUyeI+suvps0/kk9BY/Ia
Unseal Key 5: +vBOyGddh+rCPpb1cNZ4u13VgZv4tBD8h7Z0cMEu3jg8

Initial Root Token: s.HORogWa8ftxZor8f40ysXrYI

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 keys to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

The output displays the key shares and initial root key generated.

> **Important Note**: These keys are critical to both the security and the operation of Vault and should be treated as per your company's sensitive data policy.


Unseal the Vault server with the key shares until the key threshold is met:

```sh
$ kubectl exec --stdin=true --tty=true vault-0 -n vault-consul -- vault operator unseal <Unseal Key 1>

Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    1/3
Unseal Nonce       2ecd4812-c279-7de1-ca4c-bc052c5f4b0b
Version            1.9.2
Storage Type       consul
HA Enabled         true
```

```sh
$ kubectl exec --stdin=true --tty=true vault-0 -n vault-consul -- vault operator unseal <Unseal key 2>

Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    2/3
Unseal Nonce       2ecd4812-c279-7de1-ca4c-bc052c5f4b0b
Version            1.9.2
Storage Type       consul
HA Enabled         true
```

```sh
$ kubectl exec --stdin=true --tty=true vault-0 -n vault-consul -- vault operator unseal <Unseal Key 3>

Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.9.2
Storage Type    consul
Cluster Name    vault-cluster-9c0d92a1
Cluster ID      5a90da4a-4dfd-bff9-eaa7-bb7c9b74009f
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
Active Since    2022-04-21T11:41:24.062777848Z

```

> Repeat the unseal process for all Vault server pods. When all Vault server pods are unsealed they report `READY 1/1`.

```sh
$ kubectl get pods --selector='app.kubernetes.io/name=vault' -n vault-consul
NAME                                    READY   STATUS    RESTARTS   AGE
vault-0                                 1/1     Running   0          6m49s
vault-1                                 1/1     Running   0          6m49s
vault-2                                 1/1     Running   0          6m49s
```

## References

- [kubernetes-raft-deployment-guide](https://learn.hashicorp.com/tutorials/vault/kubernetes-raft-deployment-guide?in=vault/kubernetes)

## TODO

- Enable TLS
- Use GCP KMS for auto unsealing
