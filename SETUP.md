# GitLab CI/CD Setup Guide
## React + Node.js + PostgreSQL → Minikube

---

## 1. Required CI/CD Variables
Go to **GitLab → Project → Settings → CI/CD → Variables** and add:

| Variable            | Value                               | Protected | Masked |
|---------------------|-------------------------------------|-----------|--------|
| `KUBE_CONFIG_B64`   | base64-encoded kubeconfig (see §2)  | ✅        | ✅     |
| `CI_REGISTRY_USER`  | Auto-set by GitLab                  | —         | —      |
| `CI_REGISTRY_PASSWORD` | Auto-set by GitLab              | —         | —      |

---

## 2. Encode Your Minikube kubeconfig

```bash
# Start Minikube
minikube start

# Export the kubeconfig and encode it
cat ~/.kube/config | base64 -w 0
# Paste the output as the KUBE_CONFIG_B64 variable in GitLab
```

> ⚠️  If Minikube uses `127.0.0.1` as the server address,
>     replace it with your machine's LAN IP so GitLab runners can reach it.
>
> ```bash
> MINIKUBE_IP=$(minikube ip)
> sed "s/127.0.0.1/$MINIKUBE_IP/g" ~/.kube/config | base64 -w 0
> ```

---

## 3. Create Postgres Secrets (once per environment)

```bash
# Development
kubectl create secret generic postgres-secret-dev \
  --from-literal=POSTGRES_USER=appuser \
  --from-literal=POSTGRES_PASSWORD=dev_strong_password \
  --from-literal=POSTGRES_DB=appdb \
  -n dev --dry-run=client -o yaml | kubectl apply -f -

# Staging
kubectl create secret generic postgres-secret-staging \
  --from-literal=POSTGRES_USER=appuser \
  --from-literal=POSTGRES_PASSWORD=staging_strong_password \
  --from-literal=POSTGRES_DB=appdb \
  -n staging --dry-run=client -o yaml | kubectl apply -f -

# Production
kubectl create secret generic postgres-secret-prod \
  --from-literal=POSTGRES_USER=appuser \
  --from-literal=POSTGRES_PASSWORD=prod_strong_password \
  --from-literal=POSTGRES_DB=appdb \
  -n prod --dry-run=client -o yaml | kubectl apply -f -
```

---

## 4. Enable GitLab Container Registry

Go to **GitLab → Project → Settings → General → Visibility → Container Registry** and enable it.

---

## 5. Branch → Environment Mapping

| Branch    | Trigger | Environment |
|-----------|---------|-------------|
| `develop` | Auto    | Development (`dev` namespace) |
| `main`    | Manual  | Staging (`staging` namespace) |
| `main`    | Manual  | Production (`prod` namespace) — requires staging first |

---

## 6. Access the App on Minikube

```bash
# Dev
minikube service frontend -n dev

# Staging
minikube service frontend -n staging
```

---

## 7. Project Structure

```
.
├── .gitlab-ci.yml
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── src/
├── backend/
│   ├── Dockerfile
│   └── server.js
└── k8s/
    ├── postgres/
    │   └── postgres.yaml
    ├── backend/
    │   └── backend.yaml
    └── frontend/
        └── frontend.yaml
```
