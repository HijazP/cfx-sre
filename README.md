# CFX SRE Take-Home Assignment

This repo contains two minimal backend services in a monorepo:
- `go-api` (Go) — exposes `/albums`. **Ref:** https://go.dev/doc/tutorial/web-service-gin.
- `express-api` (Node.js/Express) exposes `/` and `/user/`. **Ref:** https://expressjs.com/en/starter/basic-routing.html.

---

## Repository Structure

```
.
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
└── README.md
```