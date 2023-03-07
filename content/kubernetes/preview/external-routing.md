---
Title: Establish external routing on the REC
linkTitle: External routing
description: 
weight: 9
alwaysopen: false
categories: ["Platforms"]
aliases: {
    /kubernetes/preview/external-routing.md,
    /kubernetes/preview/external-routing/,
}
---

{{<note>}} This feature is currently in preview and is not for production use. To use this feature, upgrade to the 6.2.4-1 release. {{</note>}}

Every time a Redis Enterprise database (REDB), Redis Enterprise Active-Active database (REAADB), or Redis Enterprise cluster (REC) is created, the Redis Enterprise operator, automatically creates a [service](https://kubernetes.io/docs/concepts/services-networking/service/) to allow requests to be routed to that resource. 

Redis Enterprise supports three [types of services](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) for accessing databases: `ClusterIP`, `headless`, or `LoadBalancer`.

For accessing Redis Enterprise clusters (REC), Redis Enterprise supports [types of services](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types): `ClusterIP`,`nodePort`, or `LoadBalancer`.

By default, the operator creates a `ClusterIP` type service, which exposes a cluster-internal IP and can only be accessed from within the K8s cluster. For requests to be routed from outside the K8s cluster, you need an [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) controller (or [routes](https://docs.openshift.com/container-platform/4.12/networking/routes/route-configuration.html) if you are using OpenShift).

## Install ingress controller

Redis Enterprise for Kubernetes supports three ingress controllers:
* [HAProxy](https://haproxy-ingress.github.io/)
* [NGINX](https://kubernetes.github.io/ingress-nginx/)
* [Istio]({{<relref "/kubernetes/re-databases/ingress_routing_with_istio.md">}}) [routes]

OpenShift users can use ({{<relref "/kubernetes/re-databases/routes.md">}}) instead of a ingress controller.

Install your chosen ingress controller, making sure `ssl-passthrough` is enabled. `ssl-passthrough` is disabled by default for NGINX but enabled by default for HAProxy.

## Configure DNS

1. Choose the hostname (FQDN) you will use to access your database according to the recommended naming conventions below, replacing `<placeholders>` with your own values.

   REC API hostname: `api-<rec-name>-<rec-namespace>.<subdomain>`
   REDB or REAADB hostname: `*-db-<rec-name>-<rec-namespace>.<subdomain>`
     We recommend using a wildcard (`*`) in place of the database name, followed by the hostname suffix.

1. Retrieve the `EXTERNAL-IP` of your ingress controller's `LoadBalancer` service.

    ``` sh
    $ kubectl get svc <haproxy-ingress | ingress-ngnix-controller> \
                        -n <ingress-ctrl-namespace>
    ```

    Below is example output for an HAProxy ingress controller running on a K8s cluster hosted by AWS.  

    ``` sh
    NAME              TYPE           CLUSTER-IP    EXTERNAL-IP                                                              PORT(S)                      AGE   
    haproxy-ingress   LoadBalancer   10.43.62.53   a56e24df8c6173b79a63d5da54fd9cff-676486416.us-east-1.elb.amazonaws.com   80:30610/TCP,443:31597/TCP   21m
    ```

1. Create DNS records to resolve your chosen REC API hostname and database hostname to the `EXTERNAL-IP` found in the previous step.

## Configure external routing on the REC

### Enable alpha features

Edit the Redis operator configmap (`operator-environment-config`) to set the alpha features flag to "true". 

```sh
  kubectl patch cm  operator-environment-config --type merge --patch "{\"data\": \
    {\"ENABLE_ALPHA_FEATURES\":\"true\"}}"
```

### Edit the REC spec

Edit the REC spec to add the `ingressOrRouteSpec` field, replacing `<placeholders>` with your own values.

Define the REC API hostname (`apiFqdnUrl`), database hostname suffix (`dbFqdnSuffix`) you chose when configuring DNS.

Add the annotations for your ingress controller and set `ssl-passthrough` to "true". 

```sh
kubectl patch rec  <rec-name> --type merge --patch "{\"spec\": \
    {\"ingressOrRouteSpec\": \
      {\"apiFqdnUrl\": \"api-<rec-name>-<rec-namespace>.redis.com\", \
      \"dbFqdnSuffix\": \"-db-<rec-name>-<rec-namespace>.redis.com\", \
      \"ingressAnnotations\": {\"kubernetes.io/ingress.class\": \"<ingress-controller>\", \"<ingress-controller>.ingress.kubernetes.io/ssl-passthrough\": \"true\"}, \
      \"method\": \"ingress\"}}}"
```