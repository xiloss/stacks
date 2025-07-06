# Harbor Integration

Please consider the following instructions to integrate the Harbor integration for idpbuilder.

To use the harbor registry integrated in the portal, it is necessary to customize the kind cluster properly, the main change is related to the insecure skip verify option for the endpoint, in fact we are not using proper signed certificates.
Create a kind-config.yaml file to achieve this:

```yaml
---
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: "kindest/node:v1.31.4"
  labels:
    ingress-ready: "true"
  extraPortMappings:
  - containerPort: 443
    hostPort: 8443
    protocol: TCP
  - containerPort: 32222
    hostPort: 32222
    protocol: TCP
containerdConfigPatches:
- |-
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."cnoe.localtest.me:8443"]
    endpoint = ["https://cnoe.localtest.me"]
  [plugins."io.containerd.grpc.v1.cri".registry.configs."cnoe.localtest.me".tls]
    insecure_skip_verify = true
  [plugins."io.containerd.grpc.v1.cri".registry.mirrors."harbor.cnoe.localtest.me"]
    endpoint = ["https://harbor.cnoe.localtest.me"]
  [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.cnoe.localtest.me".tls]
    insecure_skip_verify = true
```

Use the below command to deploy an IDP reference implementation with an Argo application that adds Harbor.

```bash
idpbuilder create \
  --use-path-routing \
  --package https://github.com/cloudstation-dev/stacks//ref-implementation \
  --package https://github.com/cloudstation-dev/stacks//harbor-integration \
  --kind-config kind-config.yaml
```

or just add the Harbor ArgoCD Application to an existing idpbuilder installation. This will only create a basic idpbuilder deployment with argocd and gitea.

```bash
idpbuilder create \
  --use-path-routing \
  --package https://github.com/cloudstation-dev/stacks//harbor-integration \
  --kind-config kind-config.yaml
```

As you see above, this add-on to `idpbuilder` has a dependency on the [reference implementation](../ref-implementation/). This command primarily does the following:

1. Installs Harbor helmchart as [ArgoCD Application](./harbor.yaml)

Once the custom package is installed, harbor can be accessed pointing to the UI at `https://harbor.cnoe.localtest.me:8443`

The Harbor initial credentials can be retrieved running

```bash
printf '%s\n' $( kubectl -n harbor get secret harbor-core -o jsonpath="{.data.HARBOR_ADMIN_PASSWORD}" | base64 -d )
```

## IMPORTANT: To have harbor responding correctly and using it, it is necessary to add a rule for iptables as following:

```bash
sudo iptables -t nat -A OUTPUT -p tcp --dport 443 -j REDIRECT --to-port 8443
```

Once the rule is set, harbor can be accessed pointing to the UI at `https://harbor.cnoe.localtest.me`
The command `docker login https://harbor.cnoe.localtest.me` will work from the host machine. 

Please consider this as a fast configuration directive, a better result can be achieved with a proper configuration of the ingress controller. This is by the way out of scope for the current step.

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
  --kind-config kind-config.yaml
```

from the current repository root directory.

