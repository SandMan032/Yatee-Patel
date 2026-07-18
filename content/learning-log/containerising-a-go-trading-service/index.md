---
title: "Containerising a Go Trading Service"
date: 2026-07-18
draft: false
description: "Notes on packaging a Go service into a small, reproducible Docker image using multi-stage builds."
summary: "A learning-log entry on multi-stage Docker builds for Go — from a 900 MB naive image down to a ~15 MB distroless artifact, and why each step matters."
tags: ["docker", "go", "devops", "learning-log"]
categories: ["Learning Log"]
projects: ["algo-trading-bot", "infra-devops"]
---

{{< alert icon="pencil" >}}
**Demo / placeholder post.** Content here is illustrative filler to showcase
formatting. It happens to be technically reasonable, but treat it as a template.
{{< /alert >}}

{{< lead >}}
A trading service should ship as one small, immutable artifact. Multi-stage
Docker builds get you there without dragging the whole Go toolchain into
production.
{{< /lead >}}

## The naive image (don't do this)

The first Dockerfile everyone writes bundles the compiler, the module cache and
the source into the final image. It works — and it's ~900 MB.

```dockerfile
FROM golang:1.25
WORKDIR /app
COPY . .
RUN go build -o bot ./cmd/bot
CMD ["./bot"]
```

## The multi-stage image (do this)

Split the build into a *builder* stage and a tiny *runtime* stage. Only the
compiled binary crosses the boundary.

```dockerfile
# ---- build stage ----
FROM golang:1.25 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download                 # cached unless deps change
COPY . .
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w" -o /out/bot ./cmd/bot

# ---- runtime stage ----
FROM gcr.io/distroless/static:nonroot
COPY --from=build /out/bot /bot
USER nonroot:nonroot
ENTRYPOINT ["/bot"]
```

{{< alert icon="check" cardColor="#1e1b4b" iconColor="#c47ad2" textColor="#e2e8f0" >}}
`CGO_ENABLED=0` plus a `distroless/static` base means **no libc, no shell, no
package manager** in the final image. Less to patch, less to attack, ~15 MB to
ship.
{{< /alert >}}

## Why the layer order matters

Copying `go.mod` / `go.sum` *before* the source lets Docker cache the dependency
download. Source changes rebuild fast; dependency changes are the only thing
that busts the cache.

![Docker image layers and how caching reuses them between builds](sample-layers.png "Placeholder: cache reuse across build layers")

The size difference, step by step:

| Stage | Base | Approx size |
| --- | --- | --- |
| Naive | `golang:1.25` | ~900 MB |
| Multi-stage + alpine | `alpine:3.20` | ~25 MB |
| Multi-stage + distroless | `distroless/static` | ~15 MB |

## A repeatable build command

```bash
# Reproducible, tagged by git SHA — no "latest" in production.
docker build \
  --build-arg VERSION="$(git rev-parse --short HEAD)" \
  -t trading-bot:"$(git rev-parse --short HEAD)" .
```

## Takeaways

1. Multi-stage builds keep the toolchain out of production.
2. Order `COPY` steps from least- to most-frequently-changing for cache hits.
3. Prefer distroless/static for a stateless Go binary — nothing to exploit.

Next in the log: wiring this image into a `docker compose` stack alongside a
local Redis and a mock broker for integration tests.
