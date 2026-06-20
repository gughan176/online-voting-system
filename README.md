# 🗳️ System Architecture: Online Voting Platform

> **Note:** This repository serves as an architectural case study and system design document. The proprietary application source code (React/Django) remains private. 

This document details the infrastructure, network security, and deployment strategy for a highly secure, containerized online voting system deployed on AWS.

## 🏗️ High-Level System Design

The application utilizes a decoupled, containerized architecture managed by **Docker Compose**, ensuring complete environment isolation.

* **Frontend:** React SPA (Single Page Application)
* **Backend:** Django REST Framework API
* **Database:** PostgreSQL
* **Gateway & Reverse Proxy:** Nginx
* **Infrastructure:** Ubuntu Linux natively hosted on AWS EC2

## 🛡️ Network Security & Isolation Model

Security is the primary directive of this system, specifically protecting voter integrity and database access.

1. **Internal Bridge Network:** The PostgreSQL database and Django backend operate on a private, isolated Docker network (`election-network`). 
2. **Database Shielding:** The database container does not expose any ports to the host machine or the public internet. It is only accessible internally by the backend service.
3. **Single Point of Entry:** Nginx acts as the sole public-facing gateway. It catches all traffic on ports 80/443 and routes it strictly based on path logic:
   * `/api/` and `/admin/` requests are securely tunneled to the Django container.
   * All other requests are routed to the React frontend.
4. **Transport Layer Security:** All public traffic is forced to HTTPS using Let's Encrypt SSL certificates managed via Certbot.

## 🐳 Containerization Strategy

The system utilizes optimized Docker builds to minimize surface area and improve performance:
* **Multi-Stage Builds:** The React frontend uses a two-stage build process. It compiles the source code in a heavy Node environment, but only the final static assets are moved into a lightweight, production-ready Nginx container.
* **Health Checks:** The orchestration strictly enforces startup orders. The backend container will not boot until the PostgreSQL container passes a rigorous `pg_isready` health check.

## 🔄 The DevOps Lifecycle

The deployment methodology follows a strict sequence to ensure zero-downtime updates and data safety:
1. **Environment Configuration:** All sensitive data (JWT Secrets, Database credentials) are injected at runtime via environment variables, completely separated from the image builds.
2. **Persistent Volumes:** Voter data is safeguarded using Docker local volumes mapped to the host, ensuring database persistence even if the containers are completely destroyed and rebuilt.
3. **Traffic Management:** Nginx configuration includes headers for `X-Real-IP` and `X-Forwarded-For` to ensure the backend can accurately log and audit client origins for fraud prevention.
