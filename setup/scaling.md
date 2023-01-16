# Scaling Coder

Coder's control plane (`coderd`) and workspaces are deployed in a Kubernetes
namespace. This document outlines vertical and horizontal scaling techniques to
ensure the `coderd` pods can accommodate user and workspace load on a Coder
deployment.

> Vertical scaling is preferred over horizontal scaling!

## Vertical Scaling

Vertical scaling or scaling up the Coder control plane (which consists of the
`coderd` pod and any additional replicas) is done by adding additional computing
resources to the `coderd` pods in the Helm chart's `values.yaml` file.

Download the values file for a deployed Coder release with the following
command:

```console
helm get values coder > values.yaml -n coder
```

Experiment with increasing CPU and memory requests and limits as the number of
workspaces in your Coder deployment increase. Pay particular attention to
whether users have their workspaces configured to auto-start at the same time
each day, which produces spike loads on the `coderd` pods. To best prevent Out
of Memory conditions aka OOM Kills, configure the memory requests and limits to
be the same Gi values. e.g., 8Gi

> Increasing `coderd` CPU and memory resources requires sufficient Kubernetes
> node machine types to accomodate `coderd`, Coder workspaces and additional
> system and 3rd party pods on the same cluster namespace.

These are example `values.yaml` resources for `coderd`'s CPU and memory for a
larger deployment with hundreds of workspaces autostarting at the same time each
day:

```yaml
coderd:
  resources:
    requests:
      cpu: "4"
      memory: "8Gi"
    limits:
      cpu: "8"
      memory: "8Gi"
```

Leading indicators of undersized `coderd` pods include users experiencing
disconnects in the web terminal, a web IDE like code-server or slowness within
the Coder UI dashboard. One condition that may be occuring is an OOM Kill where
one or more `coderd` pod fails, restarts or fails to restart and enteres a
CrashLoopBackOff status. If `coderd` restarts and there are active workspaces
and user sessions, they will be reconnected to a new `coderd` pod causing a
disconnect situation. As a Kubernetes administrator, you can also notice
restarts by noticing frequently changing and low `AGE` column when getting the
pods:

```console
kubectl get pods -n coder | grep coderd
```

## Horizontal Scaling

Another way to distribute user and workspace load on a Coder deployment is to
add additional `coderd` pods.

```yaml
coderd:
  replicas: 3
```

Coder load balances user and workspace requests across the `coderd` replicas
ensuring sufficient resources and response time.

> There is not a linear relationship between nodes and `coderd` replicas so
> experiment with incrementing replicas as you increase nodes. e.g., 8 nodes and
> 3 `coderd` replicas.

### Horizontal Pod Autoscaling

Horizontal Pod Autoscaling (HPA) is another Kubernetes technique to
automatically add, and remove, additional `coderd` pods when the existing pods
exceed sustained CPU and memory thresholds. Consult
[Kubernetes HPA documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
for the various API version implementations of HPA.
