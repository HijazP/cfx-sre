# CFX SRE Take-Home Assignment

This repo contains two minimal backend services in a monorepo:
- `go-api` (Go) — exposes `/albums`. **Ref:** https://go.dev/doc/tutorial/web-service-gin.
- `express-api` (Node.js/Express) exposes `/` and `/user/`. **Ref:** https://expressjs.com/en/starter/basic-routing.html.

---

## Repository Structure

```
.
├── services
│   ├── go-api
│   │   ├── go.mod
│   │   ├── go.sum
│   │   └── main.go
│   └── express-api
│       ├── app.js
│       ├── package-lock.json
│       └── package.json
├── .gitignore
└── README.md
```