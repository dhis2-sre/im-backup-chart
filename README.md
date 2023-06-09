# Instance Manager Backup

This chart installs a Velero schedule resource specifically tailored to back up an instance of the Instance Manager and
associated namespaces.

Please see the actual [schedule type](https://velero.io/docs/main/api-types/schedule/) and
the [value file](./charts/im-backup/values.yaml) for details.

# Install

There are multiple ways to install helm charts. Currently supported from this repository is basic Helm and Skaffold.

## Helm

The chart is published to https://dhis2-sre.github.io/im-backup-chart

To install the chart you first need to add this chart repository

```sh
helm repo add im-backup https://dhis2-sre.github.io/im-backup-chart
helm repo update
helm search repo im-backup/im-backup --versions

helm install im-backup im-backup/im-backup
```

The versions returned are gathered from [index.yaml](./index.yaml) which is
published to [this GitHub page](https://dhis2-sre.github.io/dhis2-core-helm/index.yaml).

## Skaffold

```sh
skaffold run
```

## Test backup

**WARNING: Don't execute the below commands without !**

After deploying the chart you can use the below commands to

1. Create a backup

```sh
velero backup create im-backup-test --from-schedule im-backup-dev
```

2. Delete helm releases

```sh
helm ls --all --short --namespace your-test-installation-of-the-instance-manager | xargs -L1 helm --namespace your-test-installation-of-the-instance-manager delete
```

3. Delete pvc

```sh
kubectl delete pvc --namespace your-test-installation-of-the-instance-manager --all
```

4. Delete the namespace

```sh
kubectl delete namespace your-test-installation-of-the-instance-manager
```

5. Restore the namespace from the backup

```sh
velero restore create --from-backup im-backup-test
```

## Release chart

Bump the version in [Chart.yaml](./charts/core/Chart.yaml), commit and push.

**NOTE: do not create a tag yourself!**

Our release workflow will then using [Helm chart releaser action](https://github.com/helm/chart-releaser-action)

* create a tag `im-backup-<version>`
* create a [release](https://github.com/dhis2-sre/im-backup-chart/releases) associated with the new tag
* commit an updated index.yaml with the new release
* redeploy the GitHub pages to serve the new index.yaml

Note: there might be a slight delay between the release and the `index.yaml` file being updated as GitHub pages have to
be re-deployed.
