# Custom ARC Runners

## Background

The default GitHub ARC (Actions Runner Controller) Ubuntu runners are intentionally minimal — they provide only the core GitHub runner tooling and omit many common tools required for typical CI/CD jobs.

## The runners

The images built here are intended for use with ARC runner deployments on Kubernetes — for example:

```yaml
template:
  spec:
    containers:
    - name: runner
      image: ghcr.io/ryanstanley4/arc-runners/ubuntu:latest
```
or:
```sh
helm upgrade --install "arc-runner-set" \
    --namespace "${NAMESPACE}" \
    --set githubConfigSecret='arc-secret' \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set minRunners=1 \
    --set runnerGroup="${GROUP_NAME}" \
    --set template.spec.containers[0].name=runner \
    --set template.spec.containers[0].image=ghcr.io/ryanstanley4/arc-runners/ubuntu:latest \
    --set template.spec.containers[0].command[0]=/home/runner/run.sh \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

The key `githubConfigSecret` refers to the PAT token used to authenticate, this needs to be created as a secret within the same namespace **prior** to deployment. For reference:

```sh
kubectl create secret generic arc-github-auth \
  -n github-runners \
  --from-literal=github_token='github_pat_..'
```

Note that this token needs the following org permissions:
- Administration -> Read and write (Annoying it needs this.)
- Self-hosted runners -> Read and write

---

## Runner Permissions
Once the runners are online depending on your use case it maybe neceisserry to give these permissions to your cluster. This can be done either by giving the runners full admin permissions (not ideal long term), or by creating a cluster role and binding this to the runners.

Admin:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: arc-runner-set-cluster-admin
subjects:
  - kind: ServiceAccount
    name: arc-runner-set-gha-rs-no-permission
    namespace: github-runners
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
```

role binding:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: arc-runner-deployer
rules:
  # Cluster-scoped namespace creation
  - apiGroups: [""]
    resources: ["namespaces"]
    verbs: ["get", "list", "watch", "create", "delete"]

  # Namespaced core resources
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "secrets", "serviceaccounts"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

  # Workloads
  - apiGroups: ["apps"]
    resources: ["deployments", "statefulsets", "daemonsets", "replicasets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

  # Batch jobs
  - apiGroups: ["batch"]
    resources: ["jobs", "cronjobs"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

  # Ingress (adjust for your cluster if using traefik CRDs instead)
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses", "networkpolicies"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

  # RBAC if your workflows create roles/bindings in target namespaces
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["roles", "rolebindings"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: arc-runner-deployer-binding
subjects:
  - kind: ServiceAccount
    name: arc-runner-set-gha-rs-no-permission
    namespace: github-runners
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: arc-runner-deployer
```


## Included Tools

The Ubuntu image includes the following tools (see `images/ubuntu/Dockerfile`):

- **Base image:** Ubuntu
- **GitHub Actions Runner**
- **Podman** (container runtime)
- **Golang**
- **GitHub CLI** (`gh`)
- **kubectl** (Kubernetes CLI)
- **Helm** (Kubernetes package manager)
- **curl, wget, git, unzip, jq, tar, gzip, lsb-release, build-essential, ca-certificates, software-properties-common, apt-transport-https, gnupg, libssl-dev**
- **System packages are upgraded for security and latest updates**

## Building the Image

To build the Ubuntu runner image locally:

```sh
cd images/ubuntu
podman build -t arc-runner-ubuntu:latest .
# or use docker if preferred
# docker build -t arc-runner-ubuntu:latest .
```

## Entrypoint & User

- The image runs as a non-root user (`runner`).
- The entrypoint is `/home/runner/run.sh` (ensure your deployment provides this script or overrides as needed).

## Customization

To add more tools or change versions, edit `images/ubuntu/Dockerfile` and rebuild the image.

## Contributing

Contributions are welcome! Please open issues or pull requests for improvements or additional tooling.

---

## License

This project is licensed under the MIT License.