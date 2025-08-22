# CFX SRE Take-Home Assignment

This repo contains two minimal backend services in a monorepo:
- `go-api` (Go) — exposes `/albums`. **Ref:** https://go.dev/doc/tutorial/web-service-gin.
- `express-api` (Node.js/Express) exposes `/` and `/user/`. **Ref:** https://expressjs.com/en/starter/basic-routing.html.

It includes:
- Dockerfiles for both services
- GitHub Actions CI to build & push images to Docker Hub
- Kubernetes manifests (namespace, deployments, services, optional ingress)

---

## Repository Structure

```
.
├── .github
│   └── workflows
│       └── cicd.yaml
├── k8s
│   ├── express-deployment.yaml
│   ├── express-service.yaml
│   ├── go-deployment.yaml
│   ├── go-service.yaml
│   ├── ingress.yaml
│   ├── kustomization.yaml
│   └── namespace.yaml
├── services
│   ├── express-api
│   │   ├── app.js
│   │   ├── Dockerfile
│   │   ├── package-lock.json
│   │   └── package.json
│   └── go-api
│       ├── Dockerfile
│       ├── go.mod
│       ├── go.sum
│       └── main.go
└── .gitignore      
└── README.md
```

---

## 1) Prerequisites

- Docker
- Kubernetes cluster (minikube/kind/others)
- `kubectl` installed and pointing to cluster
- (For ingress) Ingress controller (e.g., NGINX ingress)
- Container/image registry (Docker Hub/GitHub Packages/Harbor)

---

## 2) Configure CI (GitHub Actions) — **Image Registry**

This pipeline targets **GitHub Packages**. Create two secrets in GitHub repo settings:
- `GH_PKG_TOKEN` — A personal access token, generate from https://github.com/settings/tokens

Images pushed:
- ghcr.io/hijazp/go-api:latest
- ghcr.io/hijazp/nodejs-api:latest

> If you want to use another registry, or confgure other settings, just change env section in `.github/workflows/cicd` file

Workflows:
1. Checkout repository
2. Log in to the container registry (GitHub Actions)
3. Extract metadata (tags, labels) for Docker
4. Build and push Docker image

---

## 3) Local Run

Commands for get all runs

```bash
# (suppose ingress is already installed)
kubectl apply -k k8s/

# Check if all resource up/ready
kubectl get all -n apps
kubectl get ingress -n apps
```

Set mapping domain

```bash
IP=$(kubectl get ingress apps-ingress -n apps -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "$IP apps.test" | sudo tee -a /etc/hosts
```

## 4) Endpoint Test

```bash
# Express (GET /)
curl http://apps.test/
# Express (POST /)
curl -X POST http://apps.test/
# Express (PUT /user)
curl -X PUT http://apps.test/user
# Express (DELETE /user)
curl -X DELETE http://apps.test/user

# Go (GET /albums)
curl http://apps.test/albums
# Go (POST /albums)
curl -X POST http://app.test/albums -H 'Content-Type: application/json' -d '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
# Go (GET /albums/:id)
curl http://apps.test/albums/2

```

For test endpoint from public url, use https://cfx.hanifalir.me

## 5) (Bonus) Monitoring and Logging

Add monitoring and logging tools using Helm

```bash
# Monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Logging
helm install loki grafana/loki-stack \
  -n logging --create-namespace \
  -f k8s/logging/loki-values.yaml

kubectl get pods -n logging

# kube-prometheus-stack
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace \
  -f k8s/monitoring/kube-prom-values.yaml

kubectl get pods -n monitoring

# Access the Grafana
kubectl apply -f k8s/monitoring/grafana-ingress.yaml

IP=$(kubectl get ingress apps-ingress -n apps -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "$IP grafana.apps.test" | sudo tee -a /etc/hosts
```

Open `http://grafana.apps.local` on browser and login with:

```
USERNAME: admin
PASSWORD: prom-operator
```

The default monitoring dashboards is available (can be checked at http://grafana.apps.test/dashboards) and the Loki data source can be used.

Explore logs (Loki):
- Select data source: Loki → Log browser.
- Quick filter:
    - namespace = "apps" → view all your application logs.
    - Add pod/container labels to narrow down.
- Example query:
    - All Express logs: {namespace="apps", app="express"}
    - Find errors in Go: {namespace="apps", app="go-albums"} |= "error"
> Note: The label: app is derived from the label in the Deployment (app:express and app:go-albums). Ensure consistency.

## 6) Cleaning up

Delete apps:

```bash
kubectl delete -k k8s
```

Delete monitoring and logging:

```bash
helm uninstall monitoring -n monitoring
helm uninstall loki -n logging
kubectl delete ns monitoring logging
```

(Optional) Delete hos entries on `/etc/hosts` for `apps.test` and `grafana.apps.test`.

---

## Improvement suggestions

- Create an automatic scripts for deploy and clean up apps.
- Create a configuration file (like .env) to make configuration easier.
- Create a deployment strategy, because CI/CD currently only applies to continuous integration.