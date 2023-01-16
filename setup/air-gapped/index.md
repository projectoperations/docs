# Air-gapped deployment

If you need increased security for your Coder deployments, you can set up an
air-gapped deployment.

To do so, you must:

- Pull all Coder deployment resources into your air-gapped environment
- Push the images to your Docker registry,
- Deploy Coder from within your air-gapped environment

> Coder's trial license does not work in an air-gapped environment. If your
> organization is interested in evaluating Coder air-gapped, please contact
> [sales@coder.com](mailto:sales@coder.com) to discuss license requirements.

## Dependencies

Before proceeding, please ensure that you've installed the following software
dependencies:

- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [helm](https://helm.sh/docs/intro/install/)

Next, configure the following items in the same network as the Kubernetes
cluster that will run Coder (we've provided links to a suggested option for each
item type, but you're welcome to use the alternatives of your choice):

- [Docker Registry](https://hub.docker.com/_/registry)
- A [DNS server](https://coredns.io) (or you can use
  [HostAliases](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/))
- A
  [certificate authority](https://github.com/activecm/docker-ca/blob/master/Dockerfile)
  or [self-signed certificates](#self-signed-certificate-for-the-registry)

## Network configuration

Coder requires several preliminary steps to be performed on your network before
you can deploy Coder. If you don't already have the following on your network,
please see our [infrastructure setup guide](infrastructure.md):

- A certificate authority
- A domain name service
- A local Docker Registry

## Version controlling your changes to the Coder install files

Throughout this article, we will suggest changes to the Helm chart that you'll
obtain from the `.tgz` that's returned when you run `helm pull`. We recommend
version controlling your files.

## Step 1: Pull all Coder resources into your air-gapped environment

Coder is deployed through [helm](https://helm.sh/docs/intro/install/), and the
platform images are hosted in Coder's Docker Hub repo.

1. Download the Coder Helm charts by running the following command _outside_ of
   the air-gapped environment:

   ```console
   helm repo add coder https://helm.coder.com
   helm pull coder/coder
   ```

   These commands will add Coder's helm charts and pull the latest stable
   release into a tarball file whose name uses the following format:
   `coder-X.Y.Z.tgz` (X.Y.Z is the Coder release number).

1. Pull the images for the Coder platform from the following Docker Hub
   locations:

   [coder-service](https://hub.docker.com/r/coderenvs/coder-service)

   [envbox](https://hub.docker.com/r/coderenvs/envbox)

   **Optional:** [timescale](https://hub.docker.com/r/coderenvs/timescale)

   > Timescale is an internal database meant for evaluation deployments. It is
   > not It is not recommended to run this service in production. Connect to an
   > external Postgres database for production deployments.

   You can pull each of these images from their `coderenvs/<img-name>:<version>`
   registry location using the image's name and Coder version:

   ```console
   docker pull coderenvs/coder-service:<version>
   ```

   The following images are optional, though you're welcome to take advantage of
   Coder's versions instead of building your own:

   [Open VSX](https://github.com/orgs/eclipse/packages/container/package/openvsx-server)

   [enterprise-node](https://hub.docker.com/r/codercom/enterprise-node)

   [enterprise-intellij](https://hub.docker.com/r/codercom/enterprise-intellij)

   [ubuntu](https://hub.docker.com/_/ubuntu)

   When building images for your environments that rely on a custom certificate
   authority, be sure to follow the
   [docs for adding certificates](../../images/tls-certificates#adding-certificates-for-coder)
   to images.

1. Tag and push all of the images that you've downloaded in the previous step to
   your internal registry; this registry must be accessible from your air-gapped
   environment. For example, to push `coder-service`:

   ```console
   docker tag coderenvs/coder-service:<version> my-registry.com/coderenvs/coder-service:<version>
   docker push my-registry.com/coderenvs/coder-service:<version>
   ```

1. If necessary, create an `offline.values.yaml` file that includes the image
   paths for each of the Coder containers and proxy configuration similar to the
   following:

   ```yaml
   coderd:
     image: my-registry.com/coderenvs/coder-service:<version>
     # Coder will use this proxy for all outbound HTTP/HTTPS connections
     # such as when checking for updated images in the image registry.
     # However, note that images are pulled from the Kubernetes container runtime,
     # and may require a different setting.
     proxy:
       http: http://proxy.internal:8888
       exempt: cluster.local

   postgres:
     default:
       image: my-registry.com/coderenvs/timescale:<version>

   envbox:
     image: my-registry.com/coderenvs/envbox:<version>
   ```

   See [configuring forward and reverse proxies] for additional information
   about Coder's support for network proxies.

   [configuring forward and reverse proxies]: ../../guides/deployments/proxy.md

1. Once all of the resources are in your air-gapped network, run the following
   to deploy Coder to your Kubernetes cluster:

   ```console
   helm install coder . --create-namespace --namespace coder --values=offline.values.yaml
   ```

   If you'd like to run this command after navigating _into_ the `coder.tgz`
   directory, you can replace the `coder.tgz` path with a period:

   ```bash
   helm install --wait --atomic --debug --namespace coder coder . \
      --set postgres.default.image=$REGISTRY_DOMAIN_NAME/coderenvs/coder-service:<version> \
      --set envbox.image=$REGISTRY_DOMAIN_NAME/coderenvs/envbox:<version> \
      --set timescale.image=$REGISTRY_DOMAIN_NAME/coderenvs/timescale:<version> \
      -f registry-cert-values.yml
   ```

1. Next, follow the [Installation](../installation.md) guide beginning with
   **step 6** to get the access URL and the temporary admin password, which
   allows you to proceed with setting up and configuring Coder.

## Extensions marketplace

Coder users in an air-gapped environment cannot access the public VS Code
marketplace. However, you can point Coder to an air-gapped instance of
[Open VSX](https://github.com/eclipse/openvsx) to serve assets to users. For
instructions on how to do this, see
[Extensions](../../admin/workspace-management/extensions.md).

Please review the [Open VSX deployment wiki] for more information on setting up
your Open VSX deployment. Note that there are several components involved,
including:

- The server application, available as the
  [openvsx-server Docker image](https://github.com/eclipse/openvsx/pkgs/container/openvsx-server)
- A PostgreSQL instance to hold the metadata of the published extensions
  - If you use just a database for storage, Open VSX stores everything as binary
    data, which can increase the storage and network throughput of the database
    considerable. As such, Open VSX recommends leveraging external storage
    (e.g., Azure Blob Storage or Google Cloud Storage)
- Elasticsearch, which Open VSX uses as the default search engine for search
  queries that originate from the web UI; this is optional, since you can either
  turn off searches or use database queries
- GitHub OAuth for user authentication

[open vsx deployment wiki]:
  https://github.com/eclipse/openvsx/wiki/Deploying-Open-VSX
