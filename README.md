# DevOps Microservices – Monorepo

This repository contains a **monorepo implementation** of a microservices system.

All backend services, frontend, and CI/CD workflows are maintained in a single repository.

Each service has its own Dockerfile, Docker Compose configuration, and dedicated CI/CD workflow.

Diagrams and screenshots are available in the [docs](docs/) directory.

---

## Project Overview

This repository reproduces the **microservices architecture originally implemented in a GitHub Organization**.

You can view the original multi-repository project here:
👉 [GitHub Organization](https://github.com/docker-microservices)

While the services, frontend, APIs, and communication patterns remain the same, this monorepo uses a **different deployment approach**:

* **All services are consolidated in a single repository**
* **Deployment occurs on a single virtual machine** using Docker containers and Docker Compose
* Each service has its **own dedicated CI/CD workflow**, triggered only when relevant files change

This setup demonstrates how the same architecture and DevOps practices can be maintained while adopting a **monorepo structure and simplified deployment workflow**.

---

## 1. Deployment Model

* All services run as **Docker containers on a single virtual machine**.
* Services communicate through a **shared external Docker network**.
* Each container is independently buildable, deployable, and restartable.
* Docker Compose is used to define and run all services.

---

## 2. CI/CD Pipeline Setup

### Project-Level Runner and Secrets

* A **self-hosted runner** is configured at the project level.
* The runner has Docker installed and SSH access to the deployment VM.
* Project-level secrets are defined in the repository settings and used in all workflows:

**Docker Hub**

* `DOCKER_USERNAME`
* `DOCKER_PASSWORD`

**Deployment VM**

* `PROD_HOST`
* `PROD_USER`
* `PROD_SSH_KEY`
* `PROD_PORT`

---

### Service Pipelines and Triggering

* Each service has its **own dedicated CI/CD workflow**.
* Workflows are triggered **only when files related to that service are modified** on the `main` branch.
* This ensures that changes to one service **do not trigger builds for unrelated services**, keeping deployments fast and targeted.

**Example trigger for a generic service:**

```yaml
on:
  push:
    branches: [ main ]
    paths:
      - 'service-directory/**'
      - '.github/workflows/service-ci-cd.yml'
```

* Replaced `service-directory` and workflow file name for each service.

---

### Pipeline Flow

Each workflow generally follows these steps:

1. **Checkout Repository** – Pull the latest code.
2. **Build Service**

   * Backend: Maven build
   * Frontend: Angular production build
3. **Build Docker Image** – Tag the image with `GITHUB_SHA`
4. **Push Image to Docker Hub**
5. **Deploy to VM**

   * SSH into the VM
   * Pull the new image
   * Update or restart the container

This approach keeps pipelines **isolated per service** while maintaining a **centralized monorepo**.

---

## 3. API Documentation

* Backend services expose Swagger UI for interactive API testing.
* Example endpoints:

  * Books Service: `http://<VM-IP>:8081/swagger-ui/index.html`
  * Authors Service: `http://<VM-IP>:8082/swagger-ui/index.html`

---

## 4. Frontend, Local DNS, and HTTPS

* Angular frontend is served via Nginx.
* Nginx acts as a reverse proxy to the API Gateway.
* Local domain names are mapped via the host machine’s `hosts` file.
* Self-signed SSL certificates provide HTTPS and redirect HTTP requests.
Here’s a compact version for your README:

## 5. SSL Setup (Self-Signed)

### 5.1. Create SSL folder:

```bash
sudo mkdir -p /etc/nginx/ssl && cd /etc/nginx/ssl
````

### 5.2. Create `local.bookstore.cnf`:

```ini
[ req ]
default_bits=2048
prompt=no
default_md=sha256
req_extensions=req_ext
distinguished_name=dn

[ dn ]
C=XX
ST=Local
L=Local
O=Local Dev
OU=Dev
CN=local.bookstore

[ req_ext ]
subjectAltName=@alt_names

[ alt_names ]
DNS.1=local.bookstore
```

### 5.3. Generate key and certificate:

```bash
sudo openssl genrsa -out local.bookstore.key 2048
sudo openssl req -x509 -nodes -days 365 \
  -key local.bookstore.key -out local.bookstore.crt \
  -config local.bookstore.cnf
```

### 5.4. Set permissions:

```bash
sudo chmod 600 local.bookstore.key
sudo chmod 644 local.bookstore.crt
```

### 5.5. Mount in Docker and reload Nginx:

```yaml
volumes:
  - /etc/nginx/ssl:/etc/nginx/ssl:ro
```
Add certificates in `bookstore-frontend/nginx.conf` file

```conf
ssl_certificate     /etc/nginx/ssl/local.bookstore.crt;
ssl_certificate_key /etc/nginx/ssl/local.bookstore.key;
```

---

## 5. Summary

This monorepo demonstrates:

* Microservices architecture with Spring Boot and Angular
* Independent CI/CD pipelines per service in a single repository
* Deployment of all services as Docker containers on a single VM
* Shared Docker network for inter-service communication
* Project-level self-hosted runner and secrets for secure deployments
* Secure local access with Nginx, HTTPS, and custom DNS

It provides a practical reference for **building and deploying multiple microservices from a single repository with service-level isolation**.
