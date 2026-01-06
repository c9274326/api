# Gthulhu API Server

This project has been refactored into a microservices architecture consisting of two main components: **Manager** and **DecisionMaker**.

## Architecture Overview

*   **Manager (Port 8080)**:
    *   Acts as the control plane.
    *   Handles User/Role management and Scheduling Strategy configuration.
    *   Persists data to MongoDB.
    *   Serves the Web UI.
*   **DecisionMaker (Port 8082)**:
    *   Runs as a DaemonSet on every K8S Node.
    *   Receives Metrics from the Scheduler.
    *   Scans `/proc` for Pod-PID mappings (requires HostPID).
    *   Issues JWT Tokens for the Scheduler.

## Quick Start (Local Development)

### Prerequisites
*   Go 1.22+
*   MongoDB (Replica Set required)
*   MicroK8s / Kubernetes Cluster (optional, for K8S integration)

### 1. Start MongoDB
```bash
cd deployment/local
docker compose -f docker-compose.infra.yaml up -d
```

### 2. Start Manager Service
```bash
# In terminal 1
cd api
go run main.go manager
```
Access Web UI at `http://localhost:8080`

### 3. Start DecisionMaker Service
```bash
# In terminal 2
cd api
go run main.go decisionmaker
```
(Note: DecisionMaker listens on port 8082 by default)

## Kubernetes Deployment

To deploy in Kubernetes, you need to build two separate images or use different commands for the same image.

### Build Images
```bash
# Manager
docker build -f Dockerfile -t gthulhu-api:latest .

# DecisionMaker
docker build -f Dockerfile.decisionmaker -t gthulhu-decisionmaker:latest .
```

### Deployment Configuration
*   **Manager**: Deployment, Service (Port 8080).
*   **DecisionMaker**: DaemonSet, Service (TargetPort 8082), HostPID: true, Volume Mounts for `/proc`.

## API Endpoints

### Manager (8080)
*   `POST /api/v1/auth/login`: User login
*   `GET /api/v1/strategies/self`: Get user's strategies
*   `POST /api/v1/strategies`: Create strategy

### DecisionMaker (8082)
*   `POST /api/v1/auth/token`: Get Scheduler Token (Public Key)
*   `POST /api/v1/metrics`: Report Metrics
*   `GET /api/v1/metrics`: Get Current Metrics
*   `GET /api/v1/pods/pids`: Get Pod-PID Mappings
