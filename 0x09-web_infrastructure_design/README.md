# WEB INFRASTRUCTURE


## Overview

This document describes a scaled-up, production-oriented web infrastructure for www.foobar.com. The goal is to move from a minimal three-server setup to a more resilient and horizontally scalable design while keeping components separated by role: load balancers, web servers, application servers, and a database cluster.

The infrastructure additions in this task are:

Add 1 server (totaling additional capacity for application or web tier)

HAProxy load-balancer configured as a cluster with the existing load balancer (active/active or active/passive)

Split components: web server, application server, and database each on their own server nodes (no mixed-role servers)

This file explains the reasons for each component, suggested implementation steps, and operational considerations.

High-level topology

2 x Load Balancers (HAProxy) — configured as a cluster for redundancy

N x Web Servers (nginx) — serve static assets and reverse-proxy to application servers

M x Application Servers (your app runtime, e.g., uWSGI/node/gunicorn) — host business logic and API endpoints

Database cluster (MySQL primary/replica or MySQL cluster / Galera) — dedicated database nodes, separate from web/app

Monitoring agents on each node and centralized logging/metrics backend

Firewalls / security groups controlling access between tiers and from the Internet

(Each component gets its own server role; no servers carry mixed roles in this design.)

Additions and why they're added

1. One additional server

Why: Provides capacity for separating concerns or increasing redundancy. For example, add another application server to increase concurrency and provide zero-downtime deployments or add another web server to spread static content traffic.

When to add: When CPU, memory, or request latency shows saturation on the existing application or web server. Add before user-visible impact occurs.

2. HAProxy load balancer configured as a cluster

Why: A single load balancer is a single point of failure. Running HAProxy on two nodes in a cluster (active/active or active/passive using VRRP/keepalived) provides resilience and failover.

Key points:

Use keepalived (VRRP) or a cloud provider’s load-balancer to provide a single virtual IP (VIP) that floats between HAProxy instances.

Configure sticky sessions only if the application cannot be made stateless; prefer stateless apps.

Health checks to backend web/app servers (HTTP/HTTPS checks).

Result: If one HAProxy node fails, the VIP moves to the peer and traffic continues.

3. Split components onto dedicated servers

Why: Separation of roles reduces contention for CPU / memory / I/O, simplifies scaling, hardens security boundaries, and makes troubleshooting easier.

Examples:

Web server (nginx): handles TLS termination (or pass-through if you prefer end-to-end encryption), serves static files, caching, and reverse-proxies requests to the application tier.

Application server: executes business logic, connects to the DB, emits application metrics.

Database server(s): persist and serve data; tuned for I/O intensive workloads, backups, and replication.

Result: You can scale each layer independently: add web servers for CDN-offload/static traffic, add app servers for business logic throughput, and scale DB with replicas or clustering for reads and HA.

Application server vs Web server — short explanation

Web server (nginx, Apache)

Efficient at serving static content (HTML, CSS, JS, images).

Handles TLS/SSL termination (optionally), HTTP connection management, buffering, gzip, caching.

Acts as a reverse proxy/load balancer for application servers.

Application server (Gunicorn, uWSGI, Node, Passenger, etc.)

Runs the application code (Python, NodeJS, Ruby, etc.).

Handles dynamic request processing, database calls, business logic, templating.

Usually not optimized for serving static files or handling TLS.

Best practice: Keep nginx in front to handle static assets, caching, and keep app servers focused on dynamic work.

Implementation notes (practical steps)

Provision the additional server (cloud VM or physical host).

HAProxy cluster

Install HAProxy on both load balancers.

Install and configure keepalived to provide a floating VIP between the two HAProxy nodes.

Configure identical HAProxy configs for backend pools and health checks.

Web & App servers

Stand up new nginx nodes for web tier; configure to reverse-proxy to app servers.

Start additional application server instances on separate nodes; register them in HAProxy backends.

Database

Keep dedicated DB server(s). If possible, convert single-writer MySQL to primary-replica with automated failover (MHA, Orchestrator, or Galera for multi-master).

Monitoring & Logging

Install monitoring agents (Prometheus node exporter, Sumo Logic collector, Datadog agent, etc.) on each server.

Configure application to export metrics (Prometheus exposition endpoint, StatsD, or push via agent).

Security

Use security groups / firewalls: only open ports required per tier (e.g., 80/443 public to LB; LB -> web on 80/443; web -> app on 8080; app -> DB on 3306 restricted).

Use mutual TLS or re-encrypt between layers if internal network cannot be fully trusted.

Operational considerations

Scaling strategy: Horizontal scaling for web/app tiers; vertical or clustering for DB depending on workload.

Deployment: Use rolling updates for app servers so requests keep flowing. Take nodes out of HAProxy pool before update.

Session management: Use a shared session store (Redis) or token-based stateless sessions to avoid sticky sessions dependency.

Backups: Regular DB backups and test restores.

Example HAProxy backend snippet (conceptual)

backend app_servers
  balance roundrobin
  option httpchk GET /health
  server app1 10.0.1.11:8080 check
  server app2 10.0.1.12:8080 check

Note: Keep configuration identical on both HAProxy nodes and manage them with configuration management (Ansible/Chef/Puppet).

Why this design improves reliability

No single point of failure at LB level (two HAProxy nodes with VIP).

Independent scaling of web/app/db tiers.

Clear security boundaries with dedicated hosts and firewall rules.

Better performance isolation — DB I/O does not compete with web process CPU.