# Custom ARC Runners - Ubuntu (with Podman & Extended Tooling)

## Background

The default GitHub ARC (Actions Runner Controller) Ubuntu runners are intentionally minimal — they provide only the core GitHub runner tooling and omit many common tools required for typical CI/CD jobs.

This repo builds a custom image for ARC runners, adding:

- **Podman** (as a drop-in Docker replacement — avoids Docker daemon dependency)
- Additional useful tooling (curl, git, etc.)
- A stable image base, so CI pipelines are consistent across updates

By providing a fully pre-built image, ARC runners can start faster and avoid the overhead of installing tooling at runtime.

---

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
#...
--set template.spec.containers[0].image='ghcr.io/ryanstanley4/arc-runners/ubuntu:latest'
```