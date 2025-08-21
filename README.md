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