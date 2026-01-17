# Argo CD app migration (same cluster, two Argo CD instances)

This example deploys a basic `Deployment` + `Service` and shows how to move the
same Argo CD `Application` from **Argo CD A** to **Argo CD B** on the **same
Kubernetes cluster**.

## Files

- `app/manifest.yaml` – basic `Deployment` + `Service` in namespace `demo`
- `applications/argocd-a-application.yaml` – app for Argo CD A
- `applications/argocd-b-application.yaml` – app for Argo CD B

## Prereqs

- One Kubernetes cluster (one `kubectx` context)
- Two Argo CD instances installed in different namespaces:
  - `argocd-a`
  - `argocd-b`
- This repo pushed to a Git remote reachable by Argo CD

## 1) Install two Argo CD instances (if not already)

```bash
kubectx <your-context>

kubectl create namespace argocd-a
kubectl apply -n argocd-a \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl create namespace argocd-b
kubectl apply -n argocd-b \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 2) Update repo URL in both Application manifests

Replace `REPLACE_WITH_YOUR_GIT_REPO_URL` in:

- `applications/argocd-a-application.yaml`
- `applications/argocd-b-application.yaml`

Example repo URL:

```
https://github.com/<you>/Devops_docs.git
```

## 3) Create the app in Argo CD A

```bash
kubectx <your-context>
kubectl apply -f applications/argocd-a-application.yaml
```

Then sync it from Argo CD A UI or CLI.

## 4) Migrate the app from Argo CD A -> Argo CD B

### Option A: apply the prepared Argo CD B app manifest

```bash
kubectx <your-context>
kubectl apply -f applications/argocd-b-application.yaml
```

### Option B: export from Argo CD A and re-apply into Argo CD B

```bash
kubectx <your-context>
kubectl -n argocd-a get applications.argoproj.io demo-app -o yaml > demo-app.yaml

# Edit demo-app.yaml:
# - set metadata.namespace to argocd-b
# - remove status and metadata fields like uid/resourceVersion/managedFields

kubectl -n argocd-b apply -f demo-app.yaml
```

## 5) Stop Argo CD A from managing it

Once Argo CD B is in control:

```bash
kubectx <your-context>
kubectl -n argocd-a delete applications.argoproj.io demo-app
```

Use `--cascade=false` if you want to keep live resources untouched.

