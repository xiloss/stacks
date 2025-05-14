# Harbor Integration

Please use the below command to deploy an IDP reference implementation with an Argo application that adds Harbor.

```bash
idpbuilder create \
  --use-path-routing \
  --package https://github.com/cloudstation-dev/stacks//ref-implementation \
  --package https://github.com/cloudstation-dev/stacks//harbor-integration
```

or just add the Harbor ArgoCD Application to an existing idpbuilder installation

```bash
idpbuilder create \
  --use-path-routing \
  --package https://github.com/cloudstation-dev/stacks//harbor-integration 
```

As you see above, this add-on to `idpbuilder` has a dependency on the [reference implementation](../ref-implementation/). This command primarily does the following:

1. Installs Harbor helmchart as [ArgoCD Application](./harbor.yaml)

Once the custom package is installed, harbor can be accessed pointing to the UI at `https://harbor.cnoe.localtest.me:8443`

The Harbor initial credentials can be retrieved running

```bash
printf '%s\n' $( kubectl -n harbor get secret harbor-core -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 -d )
```

## NOTE

The given version of harbor in the ArgoCD Application is enforced at version 1.11.1
To check the latest avaiable version of the harbor helm chart, use the following snippet:

```bash
helm repo add harbor https://helm.goharbor.io
helm repo update
helm search repo harbor --versions
helm search repo harbor --versions | grep -v NAME | head -1 | awk '{print $2}'
```

The version replacement can be done editing the [harbor.yaml](./harbor.yaml) file and deployed later updating the installation, running

```bash
idpbuilder create \
  --use-path-routing \
  --package ./harbor-integration 
```

from the current repository root directory.

