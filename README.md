# LeanIX Kubernetes Deploy Action

This action provides a standard way for Kubernetes deployments. The deployment process uses azure-cli to access the cluster and
kubernetes-shopify to roll out the manifest definition. All configuration for the deployment is provided using action input
variables. The kubernetes manifests are provided in a separate directory of the git repository (e.g. k8s-deployment).

## Usage

This action will use the injected secrets within the GitHub workflow environment to access a kubernetes cluster for deployment.
Use in advance the provided [leanix/secrets-action](https://github.com/leanix/secrets-action) to inject the required secrets.

A simple deployment definition would look like:
```yaml
- name: Use k8s deploy action
  uses: leanix/k8s-deploy-action@master
  with:
    namespace: myapp1
    environment: test
```

| input | required | default | description |
|-------|----------|---------|-------------|
|namespace|yes|-|The kubernetes namespace to use.|
|environment|yes|test|The cluster environment to use (e.g. test or prod)|
|template_directory|yes|k8s-deployment|Directory that contains the manifests to deploy. [see](#kubernetes-manifests)|
|vault_secret_keys|no|-|List of Azure Key Vault secret names to inject as k8s secrets. [see](#kubernetes-secrets-from-azure-key-vault)|
|dry_run|no|-|Set to true for dry run mode: Only validate and render the templates, not actually deploying to any cluster|
|disable_validation|no|-|Set to true for disabling the validate and render the templates option during the deployment.|

### Configure deployment target

Normally you must only define the namespace, that was provisioned for you and the action will find out, which clusters
your namespace is registered for:

* `namespace`            Namespace to use on that cluster

Sometimes it may be necessary to add one of the following optional filters to only deploy to a subset of clusters:

* `environment`          Only deploy to a specific environment (e.g. prod or test)
* `region`               Only deploy to a specific region (e.g. westeurope)
* `cluster`              Only deploy to a specific cluster (e.g. aks-westeurope-cluster-name)
* `cluster_tag`          Only deploy to a cluster with a specific tag (e.g. router)

Also, although it is highly discouraged, you can deploy two sets of manifests to the same namespace by using a "selector".
See https://github.com/Shopify/krane/tree/v0.31.1#sharing-a-namespace for more details.

* `selector`             Only consider resources having a certain set of labels when updating/pruning resources in the target namespace (e.g. k1=v1,k2=v2)

### Kubernetes Manifests

The manifests to deploy have to be provided in the defined `template_directory`.
As we use the tool kubernetes-deploy, the manifests are evaluated using ERB. For more information on that see
https://github.com/Shopify/kubernetes-deploy#using-templates-and-variables or https://en.wikipedia.org/wiki/ERuby.
The following input variables are used to configure the rendering of the manifests:

* `template_directory`   Directory where the manifests have been mounted
* `<any_name>`           Additional parameters to provide for the templates

The action will pass <any_name> input variable to the kubernetes-deploy rendering engine.
You can reference it in lower case, e.g. given the input variable "some_tag":

```
apiVersion: batch/v1beta1
kind: Pod
metadata:
   something: <%= some_tag %>
```

_Attention!_ If a file containes ERB syntax, it has to be named `*.erb`

There is a number of default variables that is available by default:

* `cluster`               The cluster that the manifest is currently being deployed to, e.g. aks-westeurope-prod-zugspitze
* `region`                The region that the manifest is currently being deployed to, e.g. westeurope
* `environment`           The environment that the manifest is currently being deployed to, e.g. prod
* `timestamp`             A unix timestamp generated when the deployment starts
* `subdomain`             To improve routing consistency we expose a subdomain defaulting to namespace-environment-region, e.g. foobar-prod-westeurope
* `cluster_version`       The kubernetes version running the cluster, e.g. 1.18.10
* `region_shortname`      The shortname of the region that the manifest is currently being deployed to, e.g. eu
* `timezone`              The timezone of the region that the manifest is currently being deployed to, e.g. Europe/Berlin
* `svc_host`              The hostname of the svc running in that region, e.g. eu-svc.leanix.net or horizon-svc.leanix.net
* `default_instance_host` The hostname of the default pathfinder instance that is running in that region, e.g. eu.leanix.net or horizon-sandbox.leanix.net

### Testing the Manifests / Dry run mode

If you just want to test your deployment but not actually run it, use the following environment variable:

* `dry_run`             Only validate and render the templates, not actually deploying to any cluster

### Kubernetes Secrets From Azure Key Vault

To avoid defining secrets as clear text in your manifests, this action is able to load secrets from Azure Key Vault and create
kubernetes secrets during deployment.

* `vault_secret_keys`   List of Azure Key Vault secret names (e.g. "secret-name-1 secret-name-2")

This will deploy a kubernetes secret like this:

```
apiVersion: v1
kind: Secret
metadata:
  name: $VAULT_SECRET_KEY
type: Opaque
data:
  value: $SECRET
```

## Requires
This action requires following GitHub actions in advance:
- [leanix/secrets-action@master](https://github.com/leanix/secrets-action)

## See also
This action uses [leanix-k8s-deploy](https://github.com/leanix/leanix-k8s-deploy).

## Copyright and license

Copyright 2020 LeanIX GmbH under the [Unlicense license](LICENSE).