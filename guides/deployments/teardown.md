# Teardown

This guide shows you how to tear down Coder and the cluster on which it is
deployed.

> These instructions help you remove infrastructure created when following our
> [Kubernetes setup tutorials](../../setup/kubernetes/index.md). They do not
> include teardown steps for any additional resources that you create. If you
> need to keep your cluster, you can run `helm uninstall coder`, which deletes
> all Coder services but retains workspaces and their associated disk space.

## Amazon Elastic Kubernetes Service (EKS)

1. Make sure you're running `eksctl` version 0.37.0 or later:

   ```console
   eksctl version
   ```

1. List all of the services in your cluster:

   ```console
   kubectl get svc --all-namespaces
   ```

1. Delete any services that have an `EXTERNAL-IP` value in your namespace:

   ```console
   kubectl delete svc <service-name>
   ```

1. Delete the cluster and its underlying nodes:

   ```console
   eksctl delete cluster --name <prod>
   ```

## Azure Kubernetes Service (AKS)

1. Make sure that the workspace variable for `RESOURCE_GROUP` is set to the one
   you want to delete in Azure:

   ```console
   echo $RESOURCE_GROUP
   ```

   If the variable is incorrect, fix it by setting it to the proper value:

   ```console
   RESOURCE_GROUP="<MY_RESOURCE_GROUP_NAME>"
   ```

1. Delete the cluster:

   ```console
   az group delete --resource-group $RESOURCE_GROUP
   ```

## Google Kubernetes Engine (GKE)

1. Ensure that the environment variables for `PROJECT_ID` and `CLUSTER_NAME` are
   set to those for the cluster you want to delete:

   ```console
   echo $PROJECT_ID
   echo $CLUSTER_NAME
   ```

   If these values are incorrect, you can fix this by providing the proper
   names:

   ```console
   PROJECT_ID="<MY_RESOURCE_GROUP_NAME>" \
   CLUSTER_NAME="<MY_CLUSTER_NAME>"
   ```

1. Delete the cluster:

   ```console
   gcloud beta container --project $PROJECT_ID clusters delete \
   $CLUSTER_NAME --zone <zone>
   ```
