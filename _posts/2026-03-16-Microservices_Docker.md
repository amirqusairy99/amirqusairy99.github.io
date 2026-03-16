---
title: "Lab: Microservices With Docker Compose"
description: "A beginner-friendly microservices starter: Nginx static frontend, Node/Express API gateway, and two Node/Express services orchestrated with Docker Compose. Great for learning request tracing, timeouts, and partial failures."
date: 2026-03-16
categories: ["Microservices", "Docker"]
tags: ["docker", "microservices"]
image: /assets/img/microservices-dashboard.png
---

# Microservices Web Application Starter

A simple starter app with:
- **Frontend**: static HTML/CSS/JS served by Nginx
- **API Gateway**: Node.js + Express
- **Users Service**: Node.js + Express
- **Products Service**: Node.js + Express
- **Orchestration**: Docker Compose

## Architecture

Frontend -> Gateway -> Users Service / Products Service

The frontend calls the gateway only. The gateway fans out requests to the two backend services.

## Run it

```bash
docker compose up --build
```

Then open:
- Frontend: http://localhost:3000
- Gateway health: http://localhost:8080/health

## Debugging A Single Request (Request ID)

The gateway generates an `X-Request-Id` for every inbound request (or reuses one you provide),
propagates it to downstream services, and includes it on responses. All services emit structured
JSON logs that include `requestId`, so you can follow a request end-to-end.

Example:

```bash
curl -i http://localhost:8080/api/dashboard
```

## Endpoints

Gateway:
- `GET /health`
- `GET /api/dashboard`
- `GET /api/users`
- `GET /api/products`

Users service:
- `GET /health`
- `GET /users`

Products service:
- `GET /health`
- `GET /products`

## Partial Failures

`GET /api/dashboard` returns `200 OK` even if one downstream service fails, with `users`/`products`
set to `null` and an `errors` object describing what failed. This makes it easy to learn how a
gateway can degrade gracefully instead of failing the whole page.

## Notes

This is a clean starter template for local development. Easy next upgrades:
- add a database per service
- add auth/JWT at the gateway
- add a message broker (RabbitMQ/Kafka)
- add service discovery / tracing / metrics
- add CI/CD and Kubernetes manifests