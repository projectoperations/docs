# Upgrade

This guide will show you how to upgrade your Coder deployment.

> Before proceeding, review the [upgrade considerations](considerations.md)
> article for information breaking charges, architecture changes, and so on.

## Recommendations

- As with any significant maintenance operation, we **strongly recommend**
  taking a snapshot of the database before upgrading. If there are upgrade
  issues, it is simpler and safer to roll back at the database level since it
  guarantees restoration of the system to a known working condition.

- We recommend updating no more than one major version at a time (e.g., we
  recommend moving from 1.28 to 1.29 only).

## Prerequisites

- If you haven't already, install [Helm](https://helm.sh/docs/intro/install/).

- Before beginning the update process, ensure that you've added the Coder Helm
  repo to your cluster. You can verify that the Coder repo has been added to
  Helm using `helm repo list`:

  ```console
  $ helm repo list
  NAME URL
  coder https://helm.coder.com
  ```

  If you don't have the Coder repo, you can add it:

  ```console
  helm repo add coder https://helm.coder.com
  ```

## For public sector deployments

Users with public sector deployments may need to obtain Coder's installation
resources from
[Big Bang](https://repo1.dso.mil/platform-one/big-bang/apps/developer-tools/coder)
(Helm charts) and
[Ironbank](https://repo1.dso.mil/dsop/coder-enterprise/coder-enterprise/coder-service)
(installation images).

> Both the Big Bang and Ironbank repositories are one release behind the latest
> version of Coder.

## Upgrade Coder

1. Retrieve the latest repository information:

   ```console
   helm repo update
   ```

1. Export your current Helm chart values into a file:

   ```console
   helm get values --namespace coder coder > current-values.yaml
   ```

   > Make sure that your values only contain the changes you want (e.g., if you
   > see references to a prior version, you may need to remove these).

1. Upgrade Coder with your new Helm chart:

   ```console
   helm upgrade coder coder/coder -n coder --version=<VERSION> --values current-values.yaml
   ```

## Downgrading

When attempting to troubleshoot Coder, you may want to roll back to an older
version of Coder. Doing so requires you to make and use a database snapshot
since the database schema will be out of date.
