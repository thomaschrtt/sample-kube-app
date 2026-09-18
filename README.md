# Sample app

This repository now contains a full **GitOps-ready Kubernetes setup** for this Laravel sample app.

If you connect this repo to Argo CD and point it to the `k8s` path, Argo CD can deploy the app + MySQL automatically.

## What was added

- `k8s/`: Kubernetes manifests (namespace, app, MySQL, storage, config, service)
- `argocd/application.yaml`: optional Argo CD `Application` manifest
- `.github/workflows/build-and-push-image.yml`: builds and pushes image to `ghcr.io/thomaschrtt/sample-kube-app:latest` on push to `main`

## 0) One-time GitHub setup (for the image)

Argo CD deploys Kubernetes manifests, but Kubernetes still needs a container image to run.

This repo now builds that image automatically with GitHub Actions.

1. Push your changes to `main`
2. Wait for workflow **Build and push app image** to succeed
3. Ensure package visibility for `ghcr.io/thomaschrtt/sample-kube-app` allows your cluster to pull it (public is easiest for learning)

## 1) Connect repo in Argo CD

You have two options:

### Option A (UI, easiest)

Create an Argo CD app with:

- **Repository**: `https://github.com/thomaschrtt/sample-kube-app.git`
- **Path**: `k8s`
- **Revision**: `main`
- **Namespace**: `sample-app`
- Enable **Auto Sync** (recommended)

### Option B (kubectl apply)

Apply:

```bash
kubectl apply -f argocd/application.yaml
```

(Argo CD must already be installed.)

## 2) First sync and check

```bash
kubectl get pods -n sample-app
kubectl get svc -n sample-app
```

To open the app locally:

```bash
kubectl port-forward -n sample-app svc/sample-app 8080:80
```

Then open http://localhost:8080

## How it works (simple explanation)

- **Argo CD watches Git**: it compares cluster state vs files in `k8s/`.
- If different, Argo CD applies YAML to make cluster match Git.
- **MySQL Deployment + PVC**: creates database pod and persistent storage.
- **Laravel Deployment**:
  - starts an init container first (`php artisan migrate`) to create DB schema
  - then starts the web container (Apache + PHP app)
- **ConfigMap** stores non-sensitive env vars.
- **Secret** stores sensitive env vars (`APP_KEY`, DB passwords).
- **Service** exposes the app inside cluster (`sample-app:80`).

## Important notes

- Current secret values are demo defaults for learning. Change them before any real usage.
- If you keep GHCR package private, create an `imagePullSecret` in the cluster and attach it to the Deployment.
- This setup is intentionally simple (single replica app + single MySQL pod) for learning.
