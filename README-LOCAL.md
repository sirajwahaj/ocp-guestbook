# Guestbook Local Quick-Start Guide

## 📋 About the Project
This project is a **Distributed Guestbook Application** designed for the "OpenShift: Distributed Guestbook with Cache" lab. 

It demonstrates a modern, 4-tier cloud-native architecture:
1.  **Frontend**: An Nginx-based web interface (served via Red Hat UBI).
2.  **Backend**: A Golang API that handles logic and data orchestration.
3.  **Database**: A PostgreSQL instance for long-term data persistence.
4.  **Cache**: A Redis instance for high-performance data caching.

The application is built using **Red Hat Universal Base Images (UBI)** to ensure it follows the strict security standards required by OpenShift, while remaining fully testable on a local machine using Podman.

## 🛠 Prerequisites
Ensure you have the following installed on your Mac:
- **Podman** (or Podman Desktop).
- **Go** (only if you want to run `go mod tidy` manually, but it's built inside the container).

## 🚀 One-Shot Deployment (Copy & Paste)

Copy the entire block below and paste it into your terminal from the root of the `ocp-guestbook` directory. It will handle the network, data directories, builds, and launches.

```bash
# 1. Setup Network and Local Storage
podman network create guestbook-net || true
mkdir -p ~/guestbook-data/pgdata ~/guestbook-data/redisdata

# 2. Build Application Images (Native Arch)
podman build -t guestbook-backend ./backend
podman build -t guestbook-frontend ./frontend

# 3. Cleanup Old Containers
podman rm -f postgres redis backend frontend || true

# 4. Run PostgreSQL (Database)
podman run -d --name postgres \
  --network guestbook-net \
  -e POSTGRESQL_USER=guestbook \
  -e POSTGRESQL_PASSWORD=password \
  -e POSTGRESQL_DATABASE=guestbook \
  -v ~/guestbook-data/pgdata:/var/lib/pgsql/data:Z \
  quay.io/fedora/postgresql-16:latest

# 5. Run Redis (Cache)
podman run -d --name redis \
  --network guestbook-net \
  -e REDIS_PASSWORD=password \
  -v ~/guestbook-data/redisdata:/var/lib/redis/data:Z \
  quay.io/kurs/redis:latest

# 6. Run Guestbook Backend
podman run -d --name backend \
  --network guestbook-net \
  -e DB_HOST=postgres \
  -e DB_USER=guestbook \
  -e DB_PASSWORD=password \
  -e REDIS_HOST=redis \
  -e REDIS_PASSWORD=password \
  localhost/guestbook-backend

# 7. Run Guestbook Frontend (Exposed on 8080)
podman run -d --name frontend \
  --network guestbook-net \
  -p 8080:8080 \
  localhost/guestbook-frontend

# 8. Success!
echo "------------------------------------------------"
echo "✅ App is launching! Visit: http://localhost:8080"
echo "------------------------------------------------"
```

## 📝 What happened behind the scenes?
1.  **Container Isolation**: Each service runs in its own isolated container.
2.  **Service Discovery**: The backend finds the database using the hostname `postgres` because they share the `guestbook-net` network.
3.  **Persistence**: Even if you delete the containers, your guestbook posts stay safe in `~/guestbook-data/pgdata` thanks to **Volume Mounting**.
4.  **Security**: We use **Non-Root UBI images** and the **SELinux `:Z` flag** to follow enterprise security best practices.

---
*For a detailed look at the errors we encountered during development and their solutions, see [portfolio.md](file:///Users/sirajulhaqwahaj/.gemini/antigravity/brain/80b948ec-7088-44a2-90cb-e8d645004c11/portfolio.md).*
