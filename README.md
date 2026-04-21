# build-tools

A Docker image pre-loaded with the tools commonly needed in CI/CD pipelines that deploy to AWS and Kubernetes.

## What's included

| Tool | Description |
|------|-------------|
| [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/) | Interact with AWS services |
| [kubectl](https://kubernetes.io/docs/reference/kubectl/) | Deploy and manage Kubernetes workloads |
| [Docker CLI + Buildx](https://docs.docker.com/buildx/working-with-buildx/) | Build and push container images |
| `curl`, `git`, `jq`, `gettext`, `unzip` | General-purpose shell utilities |

## Image

```
ghcr.io/nitmedia/build-tools:latest
```

Available tags:

| Tag | Description |
|-----|-------------|
| `latest` | Latest build from `main` |
| `main` | Tracks the `main` branch |
| `v1.2.3` / `v1.2` / `v1` | Pinned to a specific release |
| `sha-<short>` | Pinned to a specific commit |

---

## Using the image in a GitHub Actions workflow

Set the image as the `container` for any job. GitHub Actions will pull it and run all steps inside the container, giving your job access to all pre-installed tools without any additional install steps.

### Basic usage

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/nitmedia/build-tools:latest

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to EKS
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: us-east-1
        run: |
          aws eks update-kubeconfig --name my-cluster --region us-east-1
          kubectl apply -f k8s/
```

### Pin to a specific version (recommended for production)

Use a semver tag so your workflow is not affected by updates to `latest`:

```yaml
    container:
      image: ghcr.io/nitmedia/build-tools:v1.2.3
```

Or pin to an immutable commit SHA for the strictest reproducibility:

```yaml
    container:
      image: ghcr.io/nitmedia/build-tools:sha-a1b2c3d
```

### Full workflow example

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/nitmedia/build-tools:latest

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_DEFAULT_REGION: ${{ vars.AWS_REGION }}
        run: aws sts get-caller-identity

      - name: Update kubeconfig
        run: |
          aws eks update-kubeconfig \
            --name ${{ vars.CLUSTER_NAME }} \
            --region ${{ vars.AWS_REGION }}

      - name: Apply Kubernetes manifests
        run: kubectl apply -f k8s/

      - name: Wait for rollout
        run: kubectl rollout status deployment/my-app --timeout=120s
```

---

## Building locally

```bash
docker build -t build-tools .
docker run --rm -it build-tools bash
```

## Contributing

The image is rebuilt and published automatically on every push to `main` and on every `v*.*.*` tag via the included [GitHub Actions workflow](.github/workflows/github-action-build-and-deploy.yml).

To release a new version:

```bash
git tag v1.2.3
git push origin v1.2.3
```
