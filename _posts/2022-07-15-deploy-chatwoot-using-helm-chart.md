---
title:  "Deploy Chatwoot using Helm Chart"
author: "Jawed Salim"
date: 2022-07-15 09:35:58 +0600
description : "Setup chatwoot on kubernetes cluster using helm chart"
tags: [chatwoot, kubernetes]
---

Chatwoot is an open-source, self-hosted customer engagement suite. Chatwoot lets you view and manage your customer data, 
communicate with them irrespective of which medium they use, and re-engage them based on their profile.

For more info., visit site](https://www.chatwoot.com/)

## Prerequisites

- Kubernetes 1.16+
- Helm 3.1.0+
- PV provisioner support in the underlying infrastructure

The helm installation will create 3 "Persistent Volume Claims" for redis, rails and postgres. Setup up a default "Storage Class" (for automatic PV) or create 3 "Persistent Volumes" with the size of 8GB, before installing chatwoot. If the "Persistent Volume Claims" do not claim the "Persistent Volumes", leave storageClassName blank (inside the PV .yaml files).

## Create Namespace

```sh
kubectl create ns chatwoot
```

## Installing the chart

To access the chart, add the repo

```sh
$ helm repo add chatwoot https://chatwoot.github.io/charts
"chatwoot" has been added to your repositories
```

Check that you have access to the chart.

```sh
$ helm search repo chatwoot
NAME             	CHART VERSION	APP VERSION	DESCRIPTION
chatwoot/chatwoot	0.8.6        	v2.5.0     	Open-source customer engagement suite, an alter...
```

Donwload the `values.yaml` file

```sh
wget -O values.yaml https://raw.githubusercontent.com/chatwoot/charts/main/charts/chatwoot/values.yaml
```

Open the `values.yaml` file in your favourite editor and update the config options

```yaml

image:
  repository: chatwoot/chatwoot
  tag: v2.5.0
  pullPolicy: IfNotPresent

services:
  name: chatwoot
  internlPort: 3000
  targetPort: 3000
  type: ClusterIP # you can use LoadBalancer
  annotations: {}
    # For example
    #  service.beta.kubernetes.io/aws-load-balancer-type: external
    #  service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
    #  service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
..
..
ingress:
  enabled: true
  # For Kubernetes >= 1.18 you should specify the ingress-controller via the field ingressClassName
  # See https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/#specifying-the-class-of-an-ingress
  # ingressClassName: nginx
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/server-snippet: |
        underscores_in_headers on;
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: "chatwoot.<your-domain-name>"
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: chatwoot
              port:
                number: 3000
  tls:
   - secretName: chatwoot-tls
     hosts:
       - "chatwoot.<your-domain-name>"

postgresql:
  enabled: true
  nameOverride: chatwoot-postgresql
  postgresqlDatabase: chatwoot_production
  postgresqlUsername: postgres
  postgresqlPassword: postgres
  # The following variables are only used when internal PG is disabled
  # postgresqlHost: postgres
  # postgresqlPort: 5432
  # When defined the `postgresqlPassword` field is ignored
  # existingSecret: secret-name
  # existingSecretKey: postgresql-password

# ENVIRONMENT VARIABLES
env:
  FRONTEND_URL: "chatwoot.<your-domain-name>"
  ...
  ...

  # SMTP details
  SMTP_ADDRESS: "<smtp-hostname>"
  SMTP_AUTHENTICATION: plain
  SMTP_ENABLE_STARTTLS_AUTO: true
  SMTP_OPENSSL_VERIFY_MODE: none
  SMTP_PASSWORD: "<smtp-user-pass>"
  SMTP_PORT: 587
  SMTP_USERNAME: "<smtp-user-name>"
  ...
  ...

```

Install the chart with the release name `chatwoot` and use above updated `values.yaml` file:

```sh
helm install chatwoot chatwoot/chatwoot -f values.yaml -n chatwoot
```

Check status of helm release

```sh
$ helm list -n chatwoot

NAME    	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART         	APP VERSION
chatwoot	chatwoot 	1       	2022-05-24 16:46:39.470008 +0530 IST	deployed	chatwoot-0.8.6	v2.5.0
```

## Deployment Status

Chack Status of services, pods, deployments etc

```sh
$ get all -n chatwoot

NAME                                     READY   STATUS    RESTARTS      AGE
pod/chatwoot-chatwoot-postgresql-0       1/1     Running   1 (42h ago)   42h
pod/chatwoot-chatwoot-redis-master-0     1/1     Running   2 (42h ago)   42h
pod/chatwoot-chatwoot-redis-replicas-0   1/1     Running   2 (42h ago)   42h
pod/chatwoot-chatwoot-redis-replicas-1   1/1     Running   0             42h
pod/chatwoot-chatwoot-redis-replicas-2   1/1     Running   3 (42h ago)   42h
pod/chatwoot-worker-6f5c5f966b-bppv2     1/1     Running   0             40h
pod/chatwoot-worker-6f5c5f966b-psf85     1/1     Running   0             40h
pod/chatwoot-web-6c77558f59-njbm9        1/1     Running   0             40h

NAME                                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/chatwoot-chatwoot-postgresql-headless   ClusterIP   None             <none>        5432/TCP   42h
service/chatwoot-chatwoot-redis-headless        ClusterIP   None             <none>        6379/TCP   42h
service/chatwoot                                ClusterIP   10.152.183.13    <none>        3000/TCP   42h
service/chatwoot-chatwoot-redis-replicas        ClusterIP   10.152.183.185   <none>        6379/TCP   42h
service/chatwoot-chatwoot-postgresql            ClusterIP   10.152.183.116   <none>        5432/TCP   42h
service/chatwoot-chatwoot-redis-master          ClusterIP   10.152.183.29    <none>        6379/TCP   42h

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/chatwoot-worker   2/2     2            2           42h
deployment.apps/chatwoot-web      1/1     1            1           42h

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/chatwoot-worker-6f5c5f966b   2         2         2       42h
replicaset.apps/chatwoot-web-6c77558f59      1         1         1       42h

NAME                                                READY   AGE
statefulset.apps/chatwoot-chatwoot-postgresql       1/1     42h
statefulset.apps/chatwoot-chatwoot-redis-master     1/1     42h
statefulset.apps/chatwoot-chatwoot-redis-replicas   3/3     42h

```

Check ingress

```sh

$ kubect get ingress -n chatwoot
NAME       CLASS    HOSTS                     ADDRESS     PORTS     AGE
chatwoot   public   chatwoot.<your-domain-name>   127.0.0.1   80, 443   42h
```


## References

- [Deploy Chatwoot using Helm Charts](https://www.chatwoot.com/docs/self-hosted/deployment/helm-charts)
