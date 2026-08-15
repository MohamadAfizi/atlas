# ATLAS

> Self-hosted server observability and operations dashboard.

ATLAS is a personal infrastructure dashboard built to monitor the health, activity, and reliability of my self-hosted Linux server and the applications running on it.

The project started from a simple problem:

> I run multiple applications and Docker services on my own Linux server, but information about the server, containers, storage, projects, and historical performance is scattered across different commands and tools.

ATLAS brings that information into one dashboard.

The long-term goal is not only to show whether the server is running, but to build an operational history that can answer questions such as:

- Is the server healthy?
- Which Docker services are currently running?
- How much CPU, RAM, and storage are being used?
- What was the highest resource usage today?
- Why did CPU usage suddenly increase?
- When did a container become unhealthy?
- How much has storage grown over the last month?
- How reliable has the server been over the last year?
- What projects have been actively developed?
- Is self-hosting this infrastructure practical compared with using VPS services?

ATLAS will be developed progressively.

The guiding principle is:

**Observe → Record → Understand → Report → Integrate → Operate → Automate**

---

# Project Philosophy

ATLAS is built around a few rules.

## Solve a real problem

ATLAS exists because the server actually exists.

The dashboard should display real infrastructure data rather than simulated portfolio statistics.

## Read-only first

Monitoring comes before infrastructure control.

Early versions of ATLAS must not restart containers, execute shell commands, rebuild applications, or modify the host.

## History matters

A dashboard showing:

```text
CPU: 12%
```

only explains what is happening now.

ATLAS should eventually be able to explain:

```text
Current CPU     12%
Average         17%
Peak            91%
Peak Time       02:14
```

Historical data is therefore a core part of the project.

## Never pretend to know

If ATLAS cannot determine the current state of the server, it should display:

```text
UNKNOWN
```

rather than incorrectly reporting:

```text
HEALTHY
```

Similarly, correlation does not automatically mean causation.

If a CPU spike occurs while a container is highly active, ATLAS may identify it as a possible contributor but should not claim it definitely caused the event.

## Operations come later

Infrastructure controls will only be introduced after monitoring, history, reporting, authentication, authorization, and auditing are established.

---

# Technology Stack

Initial stack:

| Layer | Technology |
|---|---|
| Backend | Laravel / PHP 8.4 |
| Web Server | Apache 2.4 |
| Database | SQLite |
| Frontend | Blade |
| Client-side | Vanilla JavaScript |
| Styling | Custom CSS |
| Charts | Chart.js |
| Runtime | Docker |
| Orchestration | Docker Compose |
| Host | Ubuntu Server |
| Public Access | Cloudflare Tunnel |
| Routing | Apache Reverse Proxy |

ATLAS intentionally starts without:

- React
- Vue
- MySQL
- Redis
- Prometheus
- Grafana
- nginx
- Kubernetes

These technologies may be introduced only if the project eventually has a real requirement for them.

---

# Deployment Architecture

ATLAS runs independently from the other hosted applications.

```text
Internet
   │
   ▼
Cloudflare Tunnel
   │
   ▼
fizzyjamal.com
   │
   ▼
Portfolio Apache
   │
   ├── /
   │      Portfolio V3
   │
   ├── /eperjawatan/
   │      ePerjawatan container
   │
   └── /atlas/
          │
          ▼
       ATLAS
```

ATLAS will have its own Docker environment:

```text
/srv/atlas/
├── compose.yaml
├── Dockerfile
├── src/
└── data/
    └── atlas.sqlite
```

The SQLite database should remain outside the application source so that container rebuilds do not remove monitoring history.

---

# Internal Architecture

ATLAS should eventually separate the public dashboard from infrastructure collection.

```text
                  Linux Host
                      │
              ┌───────┴───────┐
              │               │
        System Metrics    Docker Engine
              │               │
              └───────┬───────┘
                      │
                      ▼
              ATLAS Collector
                      │
                      ▼
                   SQLite
                      │
                      ▼
                 ATLAS Web
                      │
                      ▼
                  Dashboard
```

The dashboard reads collected information.

The collector is responsible for gathering infrastructure information.

This separation becomes particularly important once ATLAS eventually gains infrastructure management capabilities.

---

# Data Strategy

ATLAS handles two categories of information.

## Live Data

Represents the current state of the server.

Examples:

- CPU usage
- RAM usage
- storage usage
- system load
- server uptime
- Docker container status
- Docker health
- container CPU usage
- container memory usage

## Historical Data

Represents information ATLAS needs to remember.

Examples:

- CPU history
- RAM history
- storage growth
- Docker health transitions
- server reboot events
- incidents
- project commits
- monitoring gaps
- API health results
- power consumption
- infrastructure operations
- audit records

---

# Development Roadmap

---

# Phase 0 — Foundation

## Objective

Create a clean, isolated environment for ATLAS before implementing monitoring features.

## Repository

Create a dedicated repository:

```text
atlas
```

Suggested initial structure:

```text
atlas/
├── README.md
├── compose.yaml
├── Dockerfile
├── src/
└── data/
```

## Docker

Initial services:

```text
atlas-web
atlas-collector
```

`atlas-web`:

- Laravel
- PHP 8.4
- Apache
- serves dashboard/API
- joins shared Docker `web` network

`atlas-collector`:

- Laravel CLI
- no public port
- collects system information
- collects Docker information
- writes historical information to SQLite

## Database

Use SQLite.

Example:

```text
/srv/atlas/data/atlas.sqlite
```

Laravel:

```env
DB_CONNECTION=sqlite
DB_DATABASE=/data/atlas.sqlite
```

## Public URL

ATLAS should eventually be accessible from:

```text
https://fizzyjamal.com/atlas/
```

Routing:

```text
Cloudflare
    ↓
Portfolio Apache
    ↓
/atlas/
    ↓
atlas-web
```

## Authentication

Initial versions require a simple owner login.

Full RBAC is not required yet.

The dashboard should not expose detailed infrastructure information publicly.

## Phase 0 Complete When

- Repository exists
- Laravel runs
- Docker image builds
- SQLite works
- ATLAS container starts
- `/atlas/` routing works
- login works
- basic dashboard renders

---

# Phase 1 — Observe

## Objective

Answer one question:

> Is my Linux server okay right now?

This is ATLAS V1.

---

## System Health

Collect:

### CPU

Display:

```text
CPU
Current: 12%
```

Later:

```text
Current
Average
Peak
```

### RAM

Display:

```text
RAM
4.9 GB / 12 GB
41%
```

### Storage

Display:

```text
STORAGE
Used      742 GB
Free      818 GB
Total     1.56 TB
Usage     47%
```

### Server Uptime

Example:

```text
UPTIME
18 days 7 hours
```

### Load Average

Example:

```text
LOAD
1 min     0.31
5 min     0.42
15 min    0.38
```

### Temperature

Optional.

Only implement if reliable hardware sensor information is available.

---

# System Status

ATLAS should derive a simple overall status:

```text
HEALTHY
WARNING
CRITICAL
UNKNOWN
```

Example initial rules:

```text
CPU sustained > 80%      WARNING
RAM > 85%                WARNING
Storage > 80%            WARNING
Storage > 90%            CRITICAL
Docker unhealthy         WARNING
Collector stale          UNKNOWN
```

Thresholds may become configurable later.

A stale collector must never result in a false `HEALTHY` status.

---

# Docker Monitoring

Docker should be displayed using dashboard cards rather than only a table.

Example:

```text
┌─────────────────────────────┐
│ PORTFOLIO V3              ● │
│                             │
│ Running                     │
│ php:8.4-apache              │
│                             │
│ CPU             0.7%        │
│ RAM             84 MB       │
│ Uptime          6d 14h      │
│                             │
│ 1 / 1 containers            │
└─────────────────────────────┘
```

Applications containing multiple containers should be grouped.

Example:

```text
┌─────────────────────────────┐
│ IMMICH                    ● │
│                             │
│ HEALTHY                     │
│                             │
│ 4 / 4 containers            │
│ CPU             3.2%        │
│ RAM             1.4 GB      │
│                             │
│ Server          Healthy     │
│ Database        Healthy     │
│ Redis           Healthy     │
│ ML              Healthy     │
└─────────────────────────────┘
```

## Container Information

Collect where available:

```text
container ID
container name
Compose project
Compose service
image
status
health
created time
started time
uptime
restart count
ports
CPU usage
memory usage
memory limit
```

Do not collect secrets unnecessarily.

ATLAS should not display or store container:

- passwords
- tokens
- secrets
- sensitive environment variables

---

# Collector

Suggested initial behaviour:

```text
Live status refresh:
15 seconds

Historical snapshot:
5 minutes
```

Live data is used for the dashboard.

Historical snapshots are stored in SQLite.

The exact intervals may be adjusted after observing real server load and database growth.

---

# Initial Database Tables

Do not create every future ATLAS table immediately.

Phase 1 should start small.

## system_metrics

Suggested fields:

```text
id
collected_at

cpu_percent

load_1
load_5
load_15

memory_total_mb
memory_used_mb
memory_percent

disk_total_gb
disk_used_gb
disk_free_gb
disk_percent

uptime_seconds
```

At one snapshot every five minutes:

```text
288 rows/day
~105,000 rows/year
```

This is reasonable for SQLite.

## collector_heartbeats

Suggested fields:

```text
id
collected_at
boot_id
collector_version
```

This allows ATLAS to distinguish between:

- healthy monitoring
- collector failure
- monitoring gaps
- host reboot

## container_events

Store state transitions rather than blindly storing identical statuses.

Example:

```text
14:01    immich_server    healthy
14:17    immich_server    unhealthy
14:22    immich_server    healthy
```

---

# Phase 1 Dashboard

Keep the first version compact.

```text
ATLAS
Server Operations Dashboard
────────────────────────────────────────

SERVER HEALTH

[ CPU ] [ RAM ] [ STORAGE ] [ UPTIME ]


RESOURCE HISTORY

CPU / RAM — Last 24 Hours

      ╭────╮
──────╯    ╰────────────────────


DOCKER SERVICES

[ Portfolio V3 ] [ ePerjawatan ]
[ Immich       ] [ Immich Wife ]


RECENT EVENTS

14:22    Immich recovered
14:17    Immich unhealthy
11:03    ePerjawatan restarted
```

Do not create unnecessary pages during V1.

---

# Internal Dashboard API

Blade remains responsible for the page.

Vanilla JavaScript can retrieve small JSON payloads.

Potential endpoints:

```text
GET /api/system
GET /api/docker
GET /api/history?range=24h
GET /api/events
```

The public deployed URLs will inherit the ATLAS `/atlas/` base path.

This is not intended to become a large public REST API.

The endpoints exist to keep data collection and dashboard rendering cleanly separated.

---

# Phase 1 Completion Criteria

Phase 1 is complete when:

- CPU information is real
- RAM information is real
- storage information is real
- server uptime is real
- load information is real
- Docker information is real
- Docker cards update automatically
- Docker services are grouped correctly
- container health is displayed
- container CPU/RAM is displayed
- historical metrics are stored
- 24-hour history can be displayed
- recent container events are displayed
- collector failure can be detected
- no infrastructure control exists
- no fake metric remains

At this point:

**STOP adding features and consider ATLAS V1 complete.**

---

# Phase 2 — Record

## Objective

Answer:

> What happened to my server previously?

Introduce longer historical analysis.

Time ranges:

```text
1 hour
6 hours
24 hours
7 days
30 days
90 days
1 year
```

For each metric calculate:

```text
Current
Average
Minimum
Maximum
Peak timestamp
```

Example:

```text
CPU

Current          11%
Average          14%
Peak             87%
Peak Time        14 Aug 23:41
```

RAM:

```text
Current          42%
Average          39%
Peak             73%
```

Storage:

```text
Current          742 GB
7 Days Ago       718 GB
Growth           +24 GB
```

This phase establishes ATLAS as a historical monitoring system rather than only a live dashboard.

---

# Phase 3 — Project Intelligence

## Objective

Connect infrastructure activity with the software being developed and deployed.

Potential tables:

```text
projects
project_commits
```

Project registry:

```text
Portfolio V3
ePerjawatan
ATLAS
Taskly
other hosted projects
```

Potential project information:

```text
name
description
stack
public URL
repository
local path
Docker Compose project
current branch
deployed commit
latest repository commit
last commit timestamp
status
```

Example:

```text
ePERJAWATAN

ONLINE

Laravel 13
PHP 8.4
SQLite

Last Commit
16 Aug 2026 01:42

Current Server HEAD
166a06c

Commits This Month
18
```

---

# Deployment Difference Detection

ATLAS should eventually compare:

```text
Repository main
vs
Server HEAD
```

Example:

```text
GitHub main
a312e1d

Server HEAD
95e9dd2

WARNING
Server is 3 commits behind.
```

This helps identify deployments that were merged but not pulled/deployed to the server.

---

# ATLAS Activity Graph

Build a custom development activity graph.

The purpose is not simply to reproduce GitHub's contribution graph.

ATLAS should provide additional context.

Example:

```text
           M T W T F S S
Week 1     ░ ░ █ ▓ ░ ░ ░
Week 2     ░ ▓ █ █ █ ░ ░
Week 3     █ █ ▓ ░ ░ ░ ░
```

Selecting a day:

```text
16 AUGUST 2026

7 commits
3 projects

ePerjawatan       4
Portfolio V3      2
ATLAS             1

01:42
ePerjawatan
feat: restore ePerjawatan source

00:58
Portfolio V3
fix: deployment routing
```

Filters:

```text
Project
Year
Month
Branch
```

This creates a detailed development activity history owned by ATLAS rather than depending entirely on an external contribution graph.

---

# Phase 4 — Understand & Incidents

## Objective

Move from:

> Something happened.

toward:

> What was happening around the time it happened?

Potential tables:

```text
incidents
incident_annotations
```

Example:

```text
HIGH CPU INCIDENT

Started
02:14

Recovered
02:18

Duration
4m 12s

Peak CPU
92%

RAM at Peak
63%

Possible Contributor
immich-machine-learning
```

ATLAS may correlate:

```text
02:13    Immich ML activity increased
02:14    CPU reached 92%
02:18    CPU returned to normal
```

This does not prove causation.

ATLAS should use wording such as:

```text
Possible contributor
Related activity
Concurrent event
```

rather than claiming certainty.

---

# Incident Annotations

Allow manual explanation.

Example:

```text
SERVER OFFLINE

14 March 2027
16:03 → 18:41

Detected Event
Host reboot / monitoring interruption

Cause
Power interruption

Note
Power switch accidentally turned off.

Status
Resolved
```

Manual annotations allow real-world information to become part of the operational history.

---

# Host Availability

ATLAS runs on the same server it monitors.

This creates an important limitation:

> ATLAS cannot independently observe the server while the entire server is powered off.

ATLAS should use:

- Linux boot ID
- Linux uptime
- collector heartbeat
- boot timestamps
- container events

to infer interruptions.

Example:

```text
Collector disappears
+
same Linux boot ID
=
collector interruption


Collector disappears
+
new Linux boot ID
=
host reboot / power interruption
```

Reports should distinguish:

```text
Host Availability
Monitoring Coverage
```

True independent internet availability would eventually require an external monitoring source.

---

# Phase 5 — Reporting

## Objective

Turn collected data into evidence.

Report ranges:

```text
Daily
Weekly
Monthly
Yearly
Custom
```

Formats:

```text
PDF
CSV
```

CSV is intended for raw/structured data.

PDF is intended for human-readable reporting.

---

# Monthly Report

Example:

```text
ATLAS
SERVER HEALTH REPORT

AUGUST 2027


AVAILABILITY

Host Availability       99.97%
Monitoring Coverage     99.91%


CPU

Average                 14%
Peak                    89%
Peak Time               12 Aug 02:17


RAM

Average                 5.2 GB
Peak                    9.1 GB


STORAGE

Start                    612 GB
End                      658 GB
Growth                   +46 GB


DOCKER

Services                 12
Incidents                2
Unhealthy Duration       14m


DEVELOPMENT

Active Projects          4
Commits                  47
Active Days              18


INCIDENTS

03 Aug    Immich unhealthy
12 Aug    High CPU
```

---

# Yearly Self-Hosting Report

This is one of the long-term proof-of-concept goals.

Example:

```text
ATLAS
SELF-HOSTED INFRASTRUCTURE REPORT
2027

Tracked Period
365 days

Host Availability
99.97%

Applications Hosted
8

Containers Monitored
17

CPU Average
16%

RAM Average
5.8 / 12 GB

Storage Growth
+566 GB

Incidents
1

Longest Outage
2h 38m

Development
487 commits
143 active days
```

The report provides evidence for evaluating the practicality of running applications on personally managed infrastructure.

---

# Phase 6 — Service & API Health

## Objective

Monitor services beyond Docker state.

Examples:

```text
Portfolio
ePerjawatan
Immich
ATLAS

Apache
Cloudflare Tunnel

Application APIs
External APIs
```

Example:

```text
API HEALTH

Customer API

Status          Healthy
HTTP            200
Latency         183 ms
Last Success    12 sec ago
Failures 24h    0
```

Possible configuration:

```text
name
URL
HTTP method
expected HTTP status
timeout
check interval
```

Later:

```text
authentication
expected JSON value
response validation
```

Sensitive credentials must not be exposed through the dashboard.

---

# Backup Visibility

Before ATLAS gains permission to perform backups, it can monitor them.

Example:

```text
DATABASE BACKUP

Last Backup
16 Aug 2027 02:00

Age
6h 14m

Size
84 MB

Status
Healthy
```

This remains read-only.

---

# Phase 7 — Power Monitoring

## Objective

Measure the real electrical cost of self-hosting.

This phase requires actual measurement hardware or another trustworthy measurement source.

Do not estimate actual wall power consumption solely from CPU usage.

Potential metrics:

```text
Current Watts
Voltage
Current
Daily kWh
Monthly kWh
Average Watts
Minimum Watts
Peak Watts
```

Example:

```text
POWER

Current          31 W
Average          29 W
Lowest           23 W
Peak             74 W

Today            0.72 kWh
Month            19.8 kWh
```

Potential table:

```text
power_metrics
```

Fields:

```text
id
collected_at
watts
voltage
amps
kwh_total
```

---

# Power Correlation

Power data can later be correlated with infrastructure activity.

Example:

```text
02:14

CPU
92%

Power
67 W

Container Activity
Immich ML high
```

---

# Self-Hosting Cost Analysis

Possible configurable costs:

```text
electricity tariff
hardware purchase price
expected hardware lifetime
domain cost
other infrastructure costs
```

Example:

```text
SELF-HOSTING — 2027

Electricity          RM xxx
Domain               RM xx
Hardware Cost        RM xxx

Estimated Annual Cost
RM xxx
```

Any comparison with VPS hosting should clearly document assumptions.

---

# Phase 8 — RBAC & Demo Access

## Objective

Allow ATLAS to be safely demonstrated without exposing infrastructure controls.

Potential roles:

```text
Viewer
Operator
Owner
```

## Viewer

Can access:

```text
Dashboard
Docker Health
Server Health
Projects
History
Reports
Incidents
```

Cannot access:

```text
Infrastructure Operations
Sensitive Configuration
Credentials
Backup Operations
```

A Viewer account can eventually be used for portfolio/interview demonstrations while ATLAS remains connected to the real server.

## Owner

Full authorized access.

## Operator

Future intermediate role for selected operational actions.

---

# Phase 9 — Infrastructure Operations

## Objective

Allow controlled infrastructure actions.

This is intentionally a late phase.

Potential actions:

### Docker

```text
Start Container
Stop Container
Restart Container
```

### Deployment

```text
Docker Compose Up
Rebuild Application
```

### Services

```text
Restart Apache
Restart Cloudflare Tunnel
```

### Backup

```text
Run Database Backup
Run API/Data Backup
```

---

# Operations Security

ATLAS must never expose a generic browser-based shell such as:

```text
Command:
[________________________]
```

Infrastructure operations should use predefined actions.

Example:

```text
RESTART_CONTAINER
REBUILD_PROJECT
RESTART_APACHE
RESTART_CLOUDFLARED
BACKUP_DATABASE
```

Each operation should follow:

```text
Request
   ↓
Authorization
   ↓
Validation
   ↓
Confirmation
   ↓
Optional Step-Up Verification
   ↓
Execution
   ↓
Result
   ↓
Audit
```

---

# Step-Up Verification

Higher-impact actions may require additional verification.

Examples:

```text
Restart Container
→ confirmation

Restart Apache
→ re-authentication

Restart Cloudflare
→ re-authentication / OTP

Rebuild Deployment
→ re-authentication / OTP
```

Telegram-based verification may be explored in a later version.

---

# Operations Audit

Potential table:

```text
audit_logs
```

Example:

```text
16 Aug 2027 14:32

User
Owner

Action
Restart ePerjawatan

Verification
Step-up verification

Started
14:32:10

Completed
14:32:12

Result
SUCCESS
```

Operational actions should never silently disappear from history.

---

# Phase 10 — Automation & Notifications

## Objective

Allow ATLAS to proactively communicate important infrastructure information.

Telegram is the initial planned notification platform.

---

# Morning Server Report

Example:

```text
Good morning boss.

Server is healthy.

Docker
12 / 12 services running

RAM
5.1 / 12 GB

Storage
48% used
811 GB remaining

Yesterday Peak CPU
67%

Incidents
None
```

---

# Alerts

Examples:

```text
ATLAS ALERT

ePerjawatan has been unhealthy
for 5 minutes.
```

```text
ATLAS ALERT

Storage usage reached 85%.

234 GB remaining.
```

---

# Telegram Commands

Start with read-only commands.

Potential examples:

```text
/status
/docker
/storage
/incidents
```

Infrastructure operations through Telegram are not part of the initial automation scope.

---

# Future AI Integration

AI is not required for ATLAS to monitor the server.

ATLAS itself must collect and calculate trustworthy infrastructure information.

A future local or external AI model may be used to summarize already-structured information.

Example input:

```text
CPU Peak
87%

Normal Average
14%

Peak Time
02:14

Highest Container CPU
immich-machine-learning: 72%

Nearby Event
ML activity increased at 02:13
```

Potential generated explanation:

```text
The CPU spike coincided with increased activity from
the Immich machine-learning container.
```

AI should assist with explanation and summarization.

It should not replace deterministic monitoring.

---

# Possible Version Journey

Version numbers are not fixed requirements.

They represent the expected progression of the project.

```text
ATLAS v1.0
System & Docker Monitoring

ATLAS v1.1
Historical Metrics

ATLAS v1.2
Project Intelligence & Activity Graph

ATLAS v1.3
Incidents & Correlation

ATLAS v1.4
Reporting

ATLAS v1.5
Service & API Monitoring

ATLAS v1.6
Power & Cost Analytics

ATLAS v2.0
RBAC & Demo Access

ATLAS v2.x
Infrastructure Operations

ATLAS v3
Automation & Notifications
```

---

# Immediate Implementation Plan

Despite the long-term roadmap, development begins with a very small target.

## Step 1

Create repository.

## Step 2

Create ATLAS Docker environment.

## Step 3

Install Laravel.

## Step 4

Configure SQLite.

## Step 5

Render the initial ATLAS dashboard.

At this point, fake placeholders are acceptable:

```text
CPU       --
RAM       --
STORAGE   --
UPTIME    --
```

## Step 6

Deploy ATLAS under:

```text
fizzyjamal.com/atlas/
```

## Step 7

Read real Linux CPU information.

## Step 8

Read real RAM information.

## Step 9

Read real storage information.

## Step 10

Read real Linux uptime.

## Step 11

Connect collector to Docker.

## Step 12

Retrieve real container information.

## Step 13

Group containers by Compose project.

## Step 14

Build Docker dashboard cards.

## Step 15

Implement dashboard auto-refresh.

## Step 16

Store historical system metrics.

## Step 17

Build the first 24-hour chart.

## Step 18

Record Docker health transitions.

## Step 19

Display recent infrastructure events.

## Step 20

Test collector failure and stale-data handling.

---

# ATLAS V1 Definition of Done

V1 is complete when I can open:

```text
https://fizzyjamal.com/atlas/
```

and reliably answer:

```text
Is my server running?

How much CPU is being used?

How much RAM is being used?

How much disk space remains?

How long has the server been running?

Which Docker applications are running?

Are any containers unhealthy?

What has resource usage looked like recently?

Have any Docker health changes occurred?
```

The information must come from the real server.

No fake monitoring data should remain.

ATLAS V1 is intentionally read-only.

---

# Long-Term Vision

ATLAS should evolve from a dashboard into an operational record of the self-hosted environment.

The final system may eventually provide:

```text
OBSERVE
Server
Docker
Storage
Applications
APIs
Power

RECORD
Metrics
Events
Commits
Incidents
Availability
Backups

UNDERSTAND
Trends
Peaks
Correlations
Annotations

REPORT
Daily
Monthly
Yearly
Development Activity
Self-Hosting Cost

OPERATE
Docker
Deployments
Apache
Cloudflare
Backups

PROTECT
Authentication
RBAC
Step-Up Verification
Audit Logs

AUTOMATE
Alerts
Telegram
Daily Health Summary
Scheduled Reports
```

But every capability must be introduced progressively.

The first responsibility of ATLAS is simple:

> **Tell me whether my server is okay.**
