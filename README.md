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
    --set image.repository=ghcr.io/ryanstanley4/arc-runners/ubuntu \
    --set image.tag=latest \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
```

---

## Included Tools

The Ubuntu image includes the following tools (see `images/ubuntu/Dockerfile`):

- **Base image:** Ubuntu
- **GitHub Actions Runner**
- **Podman** (container runtime)
- **Golang**
- **kubectl** (Kubernetes CLI)
- **Helm** (Kubernetes package manager)
- **curl, wget, git, unzip, jq, tar, gzip, lsb-release, build-essential, ca-certificates, software-properties-common, apt-transport-https, gnupg, libssl-dev**

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