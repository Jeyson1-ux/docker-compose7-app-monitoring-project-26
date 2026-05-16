# docker-compose7-app-monitoring-project-26

# Docker Compose Monitoring Stack + AWS ECS/Fargate Deployment

# Introduction

This project demonstrates how modern containerized applications are built, monitored, stressed, alerted, and deployed using Docker Compose locally and AWS ECS/Fargate in the cloud.

The exercise simulates a real-world production environment where multiple services work together:

* a backend application
* a database
* monitoring systems
* alerting systems
* automated stress testing
* cloud container orchestration

The project is divided into two major parts:

| Part   | Focus                                              |
| ------ | -------------------------------------------------- |
| Part A | Local container orchestration using Docker Compose |
| Part B | Cloud container deployment using AWS ECS/Fargate   |

The objective was not only to run containers, but also to understand how production systems are monitored, scaled, secured, and managed.

---

# Project Objectives

The main objectives of this project were:

* Understand multi-container orchestration
* Learn how services communicate in containerized environments
* Monitor application performance and health
* Simulate production traffic and system stress
* Configure automated alerting systems
* Learn Docker operational workflows
* Deploy containers to AWS ECS
* Understand AWS Fargate serverless container infrastructure
* Explore cloud networking and security concepts
* Learn infrastructure lifecycle management and cleanup

---

# Technologies Used

## Containerization & Orchestration

* Docker
* Docker Compose
* AWS ECS
* AWS Fargate

---

## Backend & Database

* Flask (Python)
* PostgreSQL

---

## Monitoring & Logging

* Custom monitoring dashboard
* CloudWatch Logs
* CSV metrics logging

---

## Alerting

* AWS SES (Simple Email Service)
* Email-based monitoring alerts

---

## AWS Services

* ECS
* Fargate
* CloudFormation
* IAM
* VPC
* Security Groups
* CloudWatch

---

# Part A — Docker Compose Monitoring Stack

# What is Docker Compose?

Docker Compose is a tool used to define and manage multiple containers together using a single configuration file:

```yaml
docker-compose.yaml
```

Instead of manually starting containers one by one, Docker Compose automates:

* container creation
* networking
* volume management
* dependency handling
* environment configuration

This allows an entire application stack to be started with a single command:

```bash
docker compose up -d
```

Docker Compose is commonly used for:

* local development
* testing
* staging environments
* small-scale deployments

---

# Architecture Overview

The Docker Compose stack consisted of 5 services.

| Service          | Purpose                          |
| ---------------- | -------------------------------- |
| db               | PostgreSQL database              |
| webapp           | Flask application                |
| stress-generator | Generates traffic and stress     |
| monitor          | Collects metrics and health data |
| alert-service    | Sends email alerts               |

---

# Service Breakdown

# 1. PostgreSQL Database Service

The database container stores application data.

## Concepts Demonstrated

### Persistent Data Storage

Containers are temporary by nature. Without persistent storage, all database data would disappear whenever the container restarts.

Docker volumes solve this problem by storing data outside the container lifecycle.

---

### Health Checks

The database service included health checks to verify that PostgreSQL was fully initialized before dependent services started.

This prevents application startup failures caused by unavailable databases.

---

# 2. Flask Web Application

The Flask application served as the main backend system.

It exposed multiple endpoints used for stress testing.

## Available Endpoints

| Endpoint                | Purpose                      |
| ----------------------- | ---------------------------- |
| /                       | Main application             |
| /health                 | Health check endpoint        |
| /api/cpu-intensive      | CPU stress simulation        |
| /api/memory-intensive   | Memory stress simulation     |
| /api/database-intensive | Database load simulation     |
| /api/combined-stress    | Combined workload simulation |

---

## Health Checks

The `/health` endpoint allowed Docker and monitoring systems to determine whether the application was functioning correctly.

If the endpoint repeatedly failed:

```text
container status = unhealthy
```

Health checks are critical in production because orchestration systems use them to:

* restart failed containers
* stop routing traffic to unhealthy applications
* maintain application availability

---

# 3. Stress Generator Service

The stress generator continuously sent traffic to the Flask application.

Its purpose was to simulate real-world production traffic and push the application beyond normal operating limits.

---

## Why Stress Testing Matters

Stress testing helps engineers understand:

* system limits
* bottlenecks
* failure behavior
* performance degradation

Without stress testing, applications may fail unexpectedly in production under heavy user traffic.

---

## Types of Stress Simulated

| Stress Type        | System Impact                     |
| ------------------ | --------------------------------- |
| CPU-intensive      | High processor usage              |
| Memory-intensive   | High RAM usage                    |
| Database-intensive | Heavy DB operations               |
| Combined stress    | Multiple simultaneous bottlenecks |

---

# 4. Monitoring Service

The monitoring service continuously collected metrics from the application.

---

# What is Monitoring?

Monitoring is the process of continuously observing system behavior and resource usage.

Monitoring helps answer questions such as:

* Is the application healthy?
* Is CPU usage too high?
* Is memory usage increasing dangerously?
* Are response times slowing down?
* Is the application still reachable?

---

# Metrics Collected

The monitor tracked:

* CPU usage
* memory usage
* response time
* uptime
* health status

These metrics were stored in:

```text
container_metrics.csv
```

---

# Dashboard

The monitoring service exposed a dashboard at:

```text
http://localhost:8001
```

The dashboard visualized:

* resource usage
* container health
* response times
* recent alerts

Visualization is important because raw metrics are difficult to interpret quickly.

Charts and gauges allow engineers to identify abnormal behavior immediately.

---

# Health Monitoring

The monitoring system regularly checked:

```text
/health
```

When the application exceeded resource limits or became slow:

```text
status = unhealthy
```

This simulated how production monitoring systems detect outages.

---

# 5. Alert Service

The alert service processed monitoring alerts and sent emails through AWS SES.

---

# What is an Alert?

An alert is an automated notification generated when a monitored system exceeds predefined thresholds.

Examples:

* CPU usage too high
* response time too slow
* application unavailable
* memory exhaustion

Alerts help engineers react quickly before outages become severe.

---

# Alert Thresholds

Examples of configured thresholds:

```yaml
CPU_THRESHOLD=40
MEMORY_THRESHOLD=50
RESPONSE_TIME_THRESHOLD=1000
```

If the application exceeded these values, alerts were generated.

---

# Alert Severity Levels

Two alert severities were used:

| Severity | Meaning                 |
| -------- | ----------------------- |
| Warning  | Performance degradation |
| Critical | Application failure     |

This distinction helps prioritize incidents.

---

# Alert Buffering & Cooldowns

The alert service did not send one email per event.

Instead, alerts were:

* buffered
* grouped
* rate limited

This prevents:

```text
alert flooding
```

Without cooldowns, a failing system could generate hundreds of emails in minutes.

Production monitoring systems always include alert suppression mechanisms.

---

# AWS SES Integration

AWS SES (Simple Email Service) was used to send monitoring emails.

SES required:

* verified sender email
* verified recipient email
* AWS credentials

Environment variables stored these credentials securely inside:

```text
.env
```

---

# Environment Variables and Security

Sensitive configuration values should never be hardcoded into source code.

Examples:

* AWS keys
* passwords
* email credentials

Instead, they are injected using environment variables.

Example:

```env
AWS_ACCESS_KEY_ID=YOUR_KEY
AWS_SECRET_ACCESS_KEY=YOUR_SECRET
```

---

# Why .env Files Must Not Be Committed

`.env` files contain secrets.

If committed publicly:

* AWS accounts can be compromised
* attackers can access cloud resources
* sensitive data may leak

For this reason:

```text
.env
```

must always be added to:

```text
.gitignore
```

---

# Docker Networking Concepts

Docker Compose automatically creates an internal network.

Containers communicate using service names instead of IP addresses.

For example:

```text
webapp → db
```

This simplifies service discovery.

---

# Port Mapping

Containers run internally, but services often need external access.

Port mapping exposes container ports to the host machine.

Examples:

| Host Port | Container Purpose    |
| --------- | -------------------- |
| 8080      | Flask app            |
| 8001      | Monitoring dashboard |
| 5432      | PostgreSQL           |

This enabled browser access through:

```text
localhost:8080
localhost:8001
```

---

# Docker Volumes

Docker volumes persist shared data outside container lifecycles.

The project used shared volumes for:

* metrics logs
* alert logs
* database storage

This allowed multiple containers to access the same log files.

---

# Monitoring Observations During Stress Testing

During stress testing:

* CPU usage increased significantly
* memory usage increased
* response times slowed down
* health checks failed
* alerts were triggered

Example alerts:

```text
ALERT: Slow Response
ALERT: Application Unhealthy
```

This demonstrated how resource exhaustion affects application reliability.

---

# Docker Operational Commands Used

## Start Stack

```bash
docker compose up -d --build
```

---

## View Running Containers

```bash
docker compose ps
```

---

## View Logs

```bash
docker compose logs
```

---

## Inspect Container Health

```bash
docker inspect flask-app
```

---

## Restart Individual Service

```bash
docker compose restart stress-generator
```

---

## Stop Stack

```bash
docker compose down
```

---

# Part B — AWS ECS/Fargate Deployment

# What is Amazon ECS?

Amazon ECS (Elastic Container Service) is AWS’s managed container orchestration platform.

ECS is used to:

* deploy containers
* manage scaling
* manage networking
* maintain application availability

It provides production-grade orchestration compared to Docker Compose’s local-development focus.

---

# ECS Core Concepts

# Cluster

A cluster is a logical grouping of compute resources where containers run.

Think of it as:

```text
a container hosting environment
```

---

# Task Definition

A task definition is a blueprint describing how containers should run.

It defines:

* container image
* CPU
* memory
* ports
* networking
* logging

---

# Task

A task is a running instance of a task definition.

If the task definition is the blueprint:

```text
task = running application
```

---

# Service

An ECS service maintains the desired number of running tasks.

If a task crashes:

```text
service launches replacement task
```

This provides:

* self-healing
* high availability
* automatic recovery

---

# What is AWS Fargate?

AWS Fargate is:

```text
serverless compute for containers
```

With traditional infrastructure:

* engineers manage EC2 servers
* patch operating systems
* provision capacity
* scale infrastructure manually

Fargate removes this responsibility.

AWS automatically manages:

* servers
* operating systems
* infrastructure scaling
* runtime management

Developers only configure:

* container images
* CPU
* memory
* networking

This significantly simplifies container deployment.

---

# ECS Task Deployment

An nginx container was deployed using ECS Fargate.

Task configuration included:

| Setting         | Value     |
| --------------- | --------- |
| Launch Type     | Fargate   |
| CPU             | 0.25 vCPU |
| Memory          | 1 GB      |
| Container Image | nginx     |
| Port            | 80        |

---

# CloudWatch Logging

CloudWatch Logs was configured for centralized log collection.

---

# What is CloudWatch Logging?

CloudWatch Logs stores logs from AWS services centrally.

Instead of logs existing only inside containers:

```text
logs → CloudWatch
```

This allows:

* centralized debugging
* long-term log retention
* distributed system troubleshooting

---

# Log Groups

A log group was created:

```text
/ecs/nginx-demo
```

Creating log groups before task execution prevents task startup failures.

---

# ECS Networking Concepts

ECS tasks run inside AWS VPC networking.

---

# Security Groups

Security groups act as virtual firewalls.

Inbound HTTP access was allowed on:

```text
port 80
```

from:

```text
0.0.0.0/0
```

This allowed browser access to the nginx container.

---

# Public IP Addresses

Fargate tasks received public IP addresses so they could be accessed over the internet.

---

# Elastic Network Interfaces (ENIs)

Every Fargate task creates an ENI.

ENIs provide networking connectivity between ECS tasks and the VPC.

Because tasks depend on ENIs:

* running tasks prevent cluster deletion
* networking resources must be cleaned properly

---

# ECS Task Lifecycle

Tasks transition through several states:

| State        | Meaning                  |
| ------------ | ------------------------ |
| PROVISIONING | AWS allocating resources |
| PENDING      | Container starting       |
| RUNNING      | Task active              |
| STOPPED      | Task terminated          |

Understanding these states is important for troubleshooting deployments.

---

# Infrastructure Lifecycle Management

AWS CloudFormation manages infrastructure resources.

CloudFormation creates and deletes:

* ECS clusters
* networking resources
* IAM resources
* logging resources

---

# Dependency Chains

AWS resources depend on one another.

Example:

```text
Task
→ ENI
→ Security Group
→ VPC
```

Because of these dependencies:

* ECS clusters cannot be deleted while tasks are running
* tasks must be stopped before cleanup succeeds

This demonstrated real-world cloud infrastructure dependency management.

---

# Problems Encountered and Solutions

# ECS Cluster Deletion Failure

## Problem

Cluster deletion failed because ECS tasks were still running.

---

## Cause

AWS prevents infrastructure deletion while dependent resources remain active.

---

## Solution

Running ECS tasks were manually stopped.

After tasks entered:

```text
STOPPED
```

cluster deletion succeeded.

---

# Health Check Failures

## Problem

The Flask application became unhealthy during heavy stress testing.

---

## Cause

Resource exhaustion caused slow response times and failed health checks.

---

## Solution

Stress levels were reduced and monitoring thresholds were adjusted.

---

# Key DevOps and Cloud Engineering Takeaways

## Monitoring is Essential

Production systems must continuously report:

* health
* resource usage
* response times
* failures

Without monitoring, failures become invisible.

---

## Alerting Reduces Downtime

Automated alerts help engineers react quickly before outages become severe.

---

## Logs are Critical for Troubleshooting

Logs provide detailed visibility into system behavior and failures.

Without logs, diagnosing distributed systems becomes extremely difficult.

---

## Docker Compose vs ECS/Fargate

| Docker Compose         | ECS/Fargate                    |
| ---------------------- | ------------------------------ |
| Local development      | Production orchestration       |
| Single-machine focused | Cloud-native infrastructure    |
| Simpler setup          | Scalable and managed           |
| Limited self-healing   | Automatic recovery and scaling |

---

## Infrastructure Cleanup Matters

Unused cloud resources continue generating costs.

Proper cleanup procedures are necessary to avoid unnecessary AWS charges.

---

## Modern Cloud Systems are Interconnected

Cloud-native applications involve multiple layers working together:

* compute
* networking
* storage
* IAM permissions
* monitoring
* logging
* alerting

Understanding how these systems interact is a core DevOps and cloud engineering skill.

---

# Final Conclusion

This project provided hands-on experience with:

* multi-container applications
* monitoring systems
* observability
* automated alerting
* stress testing
* AWS ECS deployment
* serverless containers using Fargate
* cloud networking
* infrastructure lifecycle management

The exercise demonstrated how modern production systems are deployed, monitored, and maintained in real-world cloud environments.

It also highlighted the operational responsibilities involved in running reliable distributed systems, including:

* performance monitoring
* incident detection
* infrastructure cleanup
* dependency management
* centralized logging
* resource optimization

This project bridged the gap between local container development and production-grade cloud-native deployment practices.
