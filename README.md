# MERN App — Containerised

![Title Image](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot3.png)

A full-stack MERN (MongoDB, Express, React, Node.js) employee management application, containerised using Docker, Docker Compose, and Kubernetes (Minikube).

> **3016ICT Secure Development Operations — Group 7**  
> Project 2: [doananhtingithub40102/mern-app](https://github.com/doananhtingithub40102/mern-app)

---

## Table of Contents

- [Application Architecture](#application-architecture)
- [Tech Stack](#tech-stack)
- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Running with Docker Compose](#running-with-docker-compose)
- [Running with Kubernetes (Minikube)](#running-with-kubernetes-minikube)
- [Accessing the Application](#accessing-the-application)
- [Testing](#testing)

---

## Application Architecture

The application is split across five containers that communicate over a shared internal Docker network (`mern-network`):

| Container     | Image            | Role                                      | Internal Port | Exposed Port |
|---------------|------------------|-------------------------------------------|---------------|--------------|
| `frontend`    | Custom (React)   | React UI served via development server    | 3000          | 3000         |
| `backend`     | Custom (Node.js) | Express REST API                          | 5000          | 5000         |
| `mongo`       | `mongo`          | MongoDB database                          | 27017         | —            |
| `mongo-express` | `mongo-express` | Web UI for MongoDB                       | 8081          | 8081         |
| `nginx`       | Custom (Nginx)   | Reverse proxy, terminates HTTPS           | 80, 443       | 80, 443      |

The frontend communicates with the backend via Nginx (`/api` proxy pass). The backend connects to MongoDB using the `MONGO_URI` environment variable. Nginx handles all external traffic and terminates SSL.

---

## Tech Stack

**Frontend:** React, Bootstrap  
**Backend:** Node.js, Express.js  
**Database:** MongoDB  
**Proxy:** Nginx (with self-signed SSL)  
**Containerisation:** Docker, Docker Compose, Kubernetes (Minikube)

---

## Prerequisites

- Ubuntu EC2 instance (or equivalent)
- Docker and Docker Compose v2
- Git
- OpenSSL (for SSL certificate generation)
- Minikube and kubectl (for Kubernetes deployment only)

Install Docker on Ubuntu:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-v2 git
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

---

## Environment Setup

### 1. Clone the repository

```bash
git clone https://github.com/Secure-Dev-Ops-Group-7/mern-app-containerised.git
cd mern-app-containerised
```

### 2. Configure environment variables

A `.env` file is included in the repository with the required variable names. Key variables:

```env
MONGO_URI=mongodb://mongo:27017/employees
```

> `MONGO_URI` was renamed from `ATLAS_URI` in the original source since MongoDB Atlas is not used — the database runs in a container.

### 3. Generate SSL certificates

Nginx requires SSL certificates to be present before starting. Generate self-signed certificates:

```bash
mkdir -p nginx/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/nginx.key \
  -out nginx/ssl/nginx.crt \
  -subj "/CN=localhost"
```

---

## Running with Docker Compose

### Build and start all containers

```bash
docker compose up --build
```

To run in the background:

```bash
docker compose up --build -d
```

### Verify containers are running

```bash
docker compose ps
```

![5 containers](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot2.png) — terminal output of `docker compose ps` showing all 5 containers running

### View logs

```bash
docker compose logs -f
```

![log files](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot5.png) — log output showing containers started and MongoDB connected successfully

### Stop containers

```bash
docker compose down
```

---

## Running with Kubernetes (Minikube)

### 1. Set up Minikube

```bash
minikube start
eval $(minikube docker-env)
```

### 2. Build images inside Minikube's Docker environment

```bash
docker build -t frontend ./client
docker build -t backend ./server
docker build -t nginx-proxy ./nginx
```

### 3. Apply Kubernetes manifests

```bash
kubectl apply -f k8s/
```

### 4. Verify pods and services

```bash
kubectl get pods
kubectl get services
```

![pods running](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot11.png) 
— terminal output of `kubectl get pods` showing all pods running

### 5. Access the application

```bash
kubectl port-forward service/nginx-service 8443:443
```

Then open `https://localhost:8443` in a browser.

ADD SCREENSHOT HERE — terminal showing port-forward command running

---

## Accessing the Application

Once running (Docker Compose or Kubernetes), the application is accessible at:

| Interface        | URL                                      |
|------------------|------------------------------------------|
| Web App (HTTPS)  | `https://<EC2-PUBLIC-IP>`               |
| Web App (HTTP)   | `http://<EC2-PUBLIC-IP>` (redirects)    |
| Mongo Express    | `http://<EC2-PUBLIC-IP>:8081`           |

> Your browser will show a security warning for the self-signed certificate — this is expected. Proceed past it.

![log files](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot4.png) — browser showing the React frontend loaded via HTTPS (full URL visible in address bar)

![log mongo](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot6.png) — browser showing Mongo Express interface (full URL visible in address bar)

---

## Testing

### Add a record via the Web Interface

1. Open the app at `https://<EC2-PUBLIC-IP>`
2. Click **Create Record**
3. Fill in employee name, position, and level
4. Submit and confirm the record appears in the list

![new entry](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot7.png) — web interface showing a newly created employee record

### Verify data via Mongo Express

1. Open `http://<EC2-PUBLIC-IP>:8081`
2. Navigate to the `employees` database
3. Confirm the record added via the web interface appears here

![mongo record](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot6.png) — Mongo Express showing the same employee record in the database

### Add a record via Mongo Express

1. In Mongo Express, insert a new document into the `employees` collection
2. Return to the web interface and confirm the new record appears

![mongo web](https://raw.githubusercontent.com/Secure-Dev-Ops-Group-7/mern-app-containerised/main/images/Screenshot10.png) — web interface showing the record added via Mongo Express
