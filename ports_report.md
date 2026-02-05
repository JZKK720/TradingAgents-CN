# TradingAgents-CN Port Configuration Report

**Generated on:** February 5, 2026  
**Repository:** TradingAgents-CN (Fork)  
**Branch:** main  

This report details all standard ports built into the TradingAgents-CN system, including their purposes, default configurations, and sources. The system supports multiple deployment modes with different port exposure strategies.

## Overview

TradingAgents-CN uses a microservices architecture with:
- **Backend API** (FastAPI/Uvicorn)
- **Frontend** (Vue.js/Nginx)
- **Database** (MongoDB)
- **Cache** (Redis)
- **Optional Management Tools** (Redis Commander, Mongo Express)

Ports are configured through Docker Compose files, Dockerfiles, and application configuration.

## Deployment Modes

### 1. Default Docker Compose (Direct Exposure)

**File:** `docker-compose.yml`  
**Description:** Front-end and back-end services are directly exposed to the host with separate ports.

| Service | Host Port | Container Port | Purpose | Source |
|---------|-----------|----------------|---------|--------|
| Backend API | 8011 | 8000 | FastAPI REST API | [docker-compose.yml#L14-15](docker-compose.yml#L14-15), [Dockerfile.backend#L82-83](Dockerfile.backend#L82-83), [app/core/config.py#L26](app/core/config.py#L26) |
| Frontend | 3011 | 80 | Vue.js SPA served by Nginx | [docker-compose.yml#L71-72](docker-compose.yml#L71-72), [Dockerfile.frontend#L43-44](Dockerfile.frontend#L43-44), [docker/nginx.conf#L2](docker/nginx.conf#L2) |
| MongoDB | 27011 | 27017 | Database service | [docker-compose.yml#L100-101](docker-compose.yml#L100-101), [app/core/config.py#L32](app/core/config.py#L32) |
| Redis | 6311 | 6379 | Cache service | [docker-compose.yml#L129-130](docker-compose.yml#L129-130), [app/core/config.py#L59](app/core/config.py#L59) |
| Redis Commander (Optional) | 8811 | 8081 | Redis management UI | [docker-compose.yml#L155-156](docker-compose.yml#L155-156) |
| Mongo Express (Optional) | 8711 | 8081 | MongoDB management UI | [docker-compose.yml#L178-179](docker-compose.yml#L178-179) |

**Usage:** Access frontend at `http://localhost:3011`, API at `http://localhost:8011`.

### 2. Hub + Nginx Compose (Reverse Proxy)

**Files:** `docker-compose.hub.nginx.yml`, `docker-compose.hub.nginx.arm.yml`  
**Description:** Single entry point through Nginx reverse proxy on host port 8080 (container 80). Internal services communicate via Docker network.

| Service | Host Port | Container Port | Purpose | Source |
|---------|-----------|----------------|---------|--------|
| Nginx Reverse Proxy | 8080 | 80 | Unified entry point for frontend and API | [docker-compose.hub.nginx.yml#L181-183](docker-compose.hub.nginx.yml#L181-183), [docker/nginx.conf#L2](docker/nginx.conf#L2) |
| Backend API (Internal) | N/A | 8000 | FastAPI REST API (accessed via /api) | [docker-compose.hub.nginx.yml#L74-75](docker-compose.hub.nginx.yml#L74-75) |
| Frontend (Internal) | N/A | 80 | Vue.js SPA (accessed via /) | [docker-compose.hub.nginx.yml#L158-159](docker-compose.hub.nginx.yml#L158-159) |
| MongoDB | 27011 | 27017 | Database service | [docker-compose.hub.nginx.yml#L22-23](docker-compose.hub.nginx.yml#L22-23) |
| Redis | 6311 | 6379 | Cache service | [docker-compose.hub.nginx.yml#L47-48](docker-compose.hub.nginx.yml#L47-48) |

**Usage:** Access everything at `http://localhost:8080` (frontend) and `http://localhost:8080/api` (API).

### 3. ARM64 Variant

**File:** `docker-compose.hub.nginx.arm.yml`  
**Description:** Same as Hub + Nginx but optimized for ARM64 architecture.

Port configuration identical to standard Hub + Nginx mode:
- Nginx: Host 8080 → Container 80
- Backend: Internal 8000
- Frontend: Internal 80
- MongoDB: Host 27011 → Container 27017
- Redis: Host 6311 → Container 6379

## Configuration Details

### Application Configuration
- **Backend Port:** Configurable via `PORT` environment variable (default: 8000)
- **MongoDB Port:** Configurable via `MONGODB_PORT` (default: 27017)
- **Redis Port:** Configurable via `REDIS_PORT` (default: 6379)

### Docker Configuration
- **EXPOSE** directives in Dockerfiles define container ports
- **ports** in docker-compose.yml map host to container ports
- **expose** in docker-compose.yml exposes ports on Docker network only

### Nginx Configuration
- **Frontend Nginx:** Listens on port 80 inside container
- **Reverse Proxy Nginx:** Routes `/api` to backend, `/` to frontend

## Security Considerations

- Default ports are standard but should be changed in production
- Database ports (27011, 6311) are exposed for development convenience
- Consider using internal networks and not exposing databases in production
- JWT and other secrets should be properly configured

## Health Checks

All services include health checks that test connectivity on their respective ports:
- Backend: `http://localhost:8011/api/health` (direct) or `http://localhost:8080/api/health` (hub proxy)
- Frontend: `http://localhost:3011/health` (direct) or `http://localhost:8080/health` (hub proxy)
- MongoDB: MongoDB ping command
- Redis: Redis ping command

## Notes

- Frontend API base: direct mode uses `VITE_API_BASE_URL=http://localhost:8011`; hub mode should use `/api` and be accessed via `http://localhost:8080`
- CORS: direct compose now allows `http://localhost:3011`; hub mode allows `http://localhost:8080` (see .env.docker)
- Ports can be customized by modifying docker-compose files or environment variables
- ARM64 variant uses platform-specific images but same port configuration
- Management tools (Redis Commander, Mongo Express) are optional and run in separate profiles