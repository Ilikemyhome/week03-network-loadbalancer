# Week 03 - Network Load Balancer Architecture

## Overview

This project demonstrates a multi-tier network architecture using Docker Compose and Nginx. A public-facing load balancer distributes incoming traffic across two backend application instances while maintaining network isolation through separate public and private subnets.

---

# Architecture Topology Diagram

```text
                    Internet / Host
                           |
                           |
                    localhost:80
                           |
                    +-------------+
                    |    NGINX    |
                    | LoadBalancer|
                    +-------------+
                      |         |
                      |         |
       +-------------------------------+
       |        private_subnet         |
       +-------------------------------+
               |               |
               |               |
        +-------------+ +-------------+
        |  Backend 1  | |  Backend 2  |
        | Port 3000   | | Port 3000   |
        +-------------+ +-------------+

Networks:
- public_subnet
- private_subnet

Load Balancer:
- Connected to both public_subnet and private_subnet

Backends:
- Connected only to private_subnet
- No direct host port exposure
```

---

# Setup & Execution Guide

## Build and Start Containers

```bash
docker compose up --build
```

## Verify Load Balancing

```bash
for i in {1..10}; do curl localhost; echo; done
```

## Check Health Endpoint

```bash
curl localhost/healthz
```

## Stop a Backend (Failover Test)

```bash
docker stop backend1
```

## Restart Backend

```bash
docker start backend1
```

## Shut Down Environment

```bash
docker compose down
```

## Remove Containers, Networks, and Volumes

```bash
docker compose down -v
```

---

# AI Disclosure & Prompt Appendix

## AI Tool Used

- Microsoft Copilot (GPT-5)

## Prompt Submitted

Act as a senior network systems architect. I need a multi-container network topology configured using Docker Compose demonstrating isolated subnets and reverse-proxy load balancing for a web application.

The architecture must include:

1. Two isolated network subnets:
   - public_subnet
   - private_subnet

2. A load balancer:
   - Connected to both subnets
   - Exposes port 80
   - Uses round-robin load balancing

3. Two backend web applications:
   - Connected only to private_subnet
   - Return a unique server identifier
   - Include a /healthz endpoint

Provide the Docker Compose file, Nginx configuration, backend application code, and testing instructions.

## What the AI Generated Correctly

- Docker Compose topology
- Nginx load balancer configuration
- Round-robin traffic distribution
- Backend application containers
- Health-check endpoint

## Security and Networking Corrections

The AI-generated draft initially discussed exposing backend application ports directly to the host.

Potential insecure example:

```yaml
ports:
  - "3000:3000"
```

This would allow users to bypass the load balancer.

Correction applied:

```yaml
expose:
  - "3000"
```

Backend containers were restricted to the private subnet and are only reachable through the Nginx load balancer.

---

# QA Test Summary

## Load Distribution Verification

Passed

Test Command:

```bash
for i in {1..10}; do curl localhost; echo; done
```

Result:

Traffic alternated evenly between Backend 1 and Backend 2, confirming round-robin load balancing.

---

## High Availability & Failover Test

Passed

Actions:

```bash
docker stop backend1
```

Result:

Traffic continued to be served by the remaining healthy backend instance.

Actions:

```bash
docker start backend1
```

Result:

Backend 1 rejoined the active load-balancing pool.

---

## Network Isolation Test

Passed

Test:

```bash
curl localhost:3000
```

Result:

Connection refused.

This confirmed that backend services were not directly accessible from the host system and could only be reached through the load balancer.

---

## Health Check Verification

Passed

Test:

```bash
curl localhost/healthz
```

Result:

```text
load balancer healthy
```

Returned HTTP 200 OK.

---

# Key Takeaways

This project demonstrates how modern applications use load balancers and network segmentation to improve scalability, reliability, and security. The Nginx load balancer distributes requests across multiple backend services while isolated private networking prevents direct external access to application servers.