# TECHNICAL PROPOSAL
## SGI-EHA — Système de Gestion de l'Information pour l'Eau, l'Hygiène et l'Assainissement
### Democratic Republic of the Congo — National WASH Information Platform

---

> **NOTE ON LANGUAGE:** This document is drafted in English for internal review purposes. The final submission must be translated into French, as required by the Dossier de Demande de Propositions (DDP). All section titles and cross-references should be verified against the original French DDP scoring tables.

---

**Submitted by:**
<!-- ⚠️ REPLACE: Insert your firm's legal name, address, country of registration, and contact details below -->
**TechSolutions Africa SARL**
<!-- ⚠️ REPLACE: Full legal address -->
[Address Line 1], [City], [Country]
Tel: [+XXX XXX XXX XXX] | Email: [contact@example.com]
Registration No.: [XXXXXXX]

**Submitted to:**
Cellule d'Exécution des Projets – Eau (CEP-O)
Ministry of Water Resources and Electricity
Democratic Republic of the Congo

**Reference:** [DDP Reference Number]
<!-- ⚠️ REPLACE: Insert the exact reference number from the DDP cover page -->

**Date of Submission:**
<!-- ⚠️ REPLACE: Insert actual submission deadline date -->
[XX] February 2026

---

## Submission Compliance Statement

This Technical Proposal (Envelope 1) is submitted in strict conformity with the two-envelope system defined in the DDP. It contains:

- **1 original** signed hard copy
- **4 hard copies**
- **1 USB drive** containing this full dossier in searchable PDF and Word format

A separate Financial Proposal (Envelope 2) is submitted simultaneously, containing 1 original, 2 hard copies, and 1 USB drive with pricing in Excel format. Both envelopes are enclosed in a sealed outer envelope marked with project references and the mandatory notice:

> **"NE PAS OUVRIR AVANT le [XX] février 2026 à 11h00"**

---

## Table of Contents

1. [Lettre de Soumission / Submission Letter](#submission-letter)
2. [Chapter 1 — Technical Architecture (50 pts)](#chapter-1)
3. [Chapter 2 — Cybersecurity Strategy & Plan (15 pts)](#chapter-2)
4. [Chapter 3 — Personnel Competencies (20 pts)](#chapter-3)
5. [Chapter 4 — Implementation Strategy (15 pts)](#chapter-4)
6. [Annex A — Administrative Documents](#annex-a)
7. [Annex B — Past Experience References](#annex-b)
8. [Annex C — CVs of Key Experts](#annex-c)
9. [Annex D — Manufacturer Authorizations](#annex-d)

---

## Submission Letter {#submission-letter}

<!-- ⚠️ REPLACE: This letter must be printed on company letterhead and signed by an authorized representative -->

[City], [XX] February 2026

To the Attention of:
The Director, Cellule d'Exécution des Projets – Eau (CEP-O)
Ministry of Water Resources and Electricity
Democratic Republic of the Congo

**Subject: Technical Proposal — SGI-EHA National WASH Information Management System**
**Reference: [DDP Reference Number]**

Dear Director,

We, the undersigned, [TechSolutions Africa SARL], duly organized and existing under the laws of [Country of Registration], with offices at [Address], hereby submit our Technical Proposal in response to the above-referenced Dossier de Demande de Propositions.

We confirm that:

1. We have read and understood all requirements in the DDP and our proposal conforms fully thereto.
2. Our proposal remains valid for a period of [90 / 120] days from the deadline for submission.
3. We have not been, and are not currently, the subject of debarment proceedings by any international financial institution.
4. We confirm the accuracy of all statements made in our proposal and the authenticity of all supporting documents attached.

We understand that the evaluation will be conducted on a 70% Technical / 30% Financial basis and that a minimum technical score of 80% must be achieved for our financial proposal to be considered.

Yours faithfully,

<!-- ⚠️ REPLACE: Authorized signatory name, title, date, and wet signature -->
____________________________
[Name of Authorized Representative]
[Title]
[TechSolutions Africa SARL]
Date: [XX February 2026]

---

---

# CHAPTER 1 — TECHNICAL ARCHITECTURE {#chapter-1}
## Evaluation Weight: 50 Points (50%)

---

## 1.1 Executive Summary of the Proposed Architecture

The SGI-EHA platform is designed as a **sovereign, high-availability, modular digital infrastructure** for the Democratic Republic of the Congo's Water, Hygiene, and Sanitation (WASH) sector. It provides a unified national data management capability for ministries, regulators (ARSPE), provincial directorates, operators, field agents, and citizens.

The architecture rests on five foundational technical choices, each deliberate and context-driven:

| Pillar | Technology | Rationale |
| :--- | :--- | :--- |
| Application Runtime | **Phoenix on the BEAM (Elixir/OTP)** | Telecom-grade fault isolation, lightweight concurrency, self-healing supervision |
| Domain Logic | **Gleam** | Strongly typed, compile-time safe domain validation, compiled to the BEAM runtime |
| Data Layer | **PostgreSQL 16 + PostGIS 3** | Authoritative source of truth; geospatial capability; WAL replication |
| User Interfaces | **Flutter** (web portal + offline-first mobile) | Single codebase, offline-first PWA and native mobile, responsive dashboards |
| Infrastructure | **Nginx + Martin + pg_featureserv + PowerSync** | Stateless, bandwidth-aware, sovereignty-compliant, Kubernetes-free |

This produces a system that behaves as a **modular microservices platform at the domain boundary level**, while maintaining the operational simplicity and performance necessary to succeed in DRC's connectivity and staffing environment.

---

## 1.2 Architectural Model: The Nucleus and Microservice Domains

### 1.2.1 The Central Nucleus

The system is organized around a **central nucleus** that acts as the orchestration and interconnection hub. The nucleus is constituted of REST/JSON APIs that manage all interactions between the core system and functional microservice domains. It enforces:

- Uniform REST/JSON standards for all internal and external communication
- Synchronous and asynchronous communication modes per microservice
- Application-level caching (ETS) to reduce database load on hot data paths
- Role-Based Access Control (RBAC) at the API boundary

### 1.2.2 Modular Microservice Domains

The nucleus connects to six isolated, independently deployable domain modules:

| Domain | Functional Scope |
| :--- | :--- |
| **Water & Infrastructure** | Drinking water points, production volumes, network technical characteristics |
| **Sanitation & Hygiene** | Liquid waste management, fecal sludge treatment, FDAL/ODF community status |
| **Project Portfolio** | Physical, financial, and institutional progress of national WASH projects |
| **Regulation & Operators** | Benchmarking of service providers, concession management, ARSPE regulatory analysis |
| **Finance & Billing** | Sector taxes, fees, automated receipts, and financial reports |
| **Community Alerts** | Public-facing interface for citizens to report infrastructure failures |

Each domain is implemented as an isolated **OTP supervisor tree** with its own:
- Process registry and lifecycle management
- Event processor (GenServer)
- In-memory cache (ETS table)
- Task supervisor for parallel work
- Dedicated API controller surface

**A crash in one domain does not affect any other domain.** Each module restarts independently via OTP supervision strategies.

### 1.2.3 Independent Deployment Support

Each domain module supports independent deployment through:

1. **Feature flags** at the application configuration level, allowing selective module activation without redeployment of the full platform.
2. **Migration isolation**: each domain owns its own database schema namespace, making schema updates domain-scoped.
3. **API versioning**: domain APIs are versioned (`/api/v1/water/`, `/api/v1/sanitation/`, etc.) to allow independent evolution.
4. **BEAM node extraction path**: should future scale justify it, any domain supervisor tree can be extracted to a dedicated BEAM node with minimal conceptual change, following the same REST/JSON contract.

### 1.2.4 Application Caching Architecture

The system implements a **three-tier caching strategy** to achieve sub-800 ms API responses:

| Cache Tier | Implementation | Contents | Invalidation |
| :--- | :--- | :--- | :--- |
| **L1 — In-process** | ETS (Erlang Term Storage) | Hot water points, dashboard metrics, GIS metadata, rate limit counters | Write-through on domain actor update |
| **L2 — Session** | PostgreSQL `user_sessions` table + ETS read-through | Auth tokens, RBAC roles, MFA state | On logout, rotation, or expiry |
| **L3 — Async precompute** | Oban cron workers | Aggregated SDG/BI indicator summaries | Nightly refresh cycle |

ETS provides sub-microsecond in-process reads. The L3 precompute pattern means dashboard loads never trigger expensive real-time aggregations.

---

## 1.3 Full System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              CLIENTS                                     │
│  [Flutter Web Portal]   [Flutter Mobile App]   [mWater / DHIS2 APIs]   │
└───────────────┬───────────────────┬───────────────────┬─────────────────┘
                │ HTTPS/WSS         │ HTTPS             │ HTTPS
                ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         EDGE LAYER                                       │
│          Nginx (TLS termination + ModSecurity WAF + auth_request)        │
└──────────────────┬────────────────┬──────────────────┬──────────────────┘
                   │ /api /socket   │ /tiles           │ /features
                   ▼                ▼                  ▼
┌──────────────────────┐  ┌──────────────┐  ┌────────────────────┐
│  BEAM CLUSTER        │  │  GIS LAYER   │  │  OFFLINE SYNC      │
│  (3 nodes)           │  │              │  │                    │
│  Phoenix Endpoint    │  │  Martin      │  │  PowerSync         │
│  Gleam Validators    │  │  (MVT tiles) │  │  (WAL → SQLite)    │
│  OTP Domain Sups.    │  │              │  │                    │
│  ETS Cache           │  │  pg_featsrv  │  │                    │
│  Oban Background     │  │  (OGC GeoJ.) │  │                    │
│  Phoenix PubSub      │  └──────┬───────┘  └────────┬───────────┘
└────────┬─────────────┘         │                   │
         │                       │ SQL               │ WAL replication
         ▼                       ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                       │
│              PostgreSQL 16 + PostGIS 3 (Primary — DRC-hosted)           │
└──────────────────────────────────┬──────────────────────────────────────┘
                                   │ Encrypted WAL streaming
                                   ▼
                    ┌──────────────────────────────┐
                    │  DISASTER RECOVERY            │
                    │  Encrypted Private Cloud      │
                    │  (pgBackRest + snapshot DR)   │
                    └──────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                    SECURITY & OBSERVABILITY                               │
│  Wazuh SIEM  │  Suricata IDS (ML-enhanced)  │  Prometheus + Grafana     │
│  AI Log Analysis Assistant  │  OpenTelemetry + PromEx                   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 1.4 Interoperability: mWater and DHIS2 Native APIs

The platform implements **native bidirectional Push/Pull APIs** for both target external systems.

### 1.4.1 mWater Integration

| Mechanism | Details |
| :--- | :--- |
| **Pull** | `MWaterSyncWorker` (Oban, hourly cron) queries mWater REST API for updated infrastructure data, normalizes records through Gleam validators, and upserts to the Water domain. |
| **Push** | Domain event on `water:updates` PubSub topic triggers an outbound notification to mWater webhook endpoint for new DRC-side records that mWater does not yet hold. |
| **Conflict resolution** | Last-write-wins with source system timestamp; conflict log retained for audit. |
| **Error handling** | Failed sync attempts are retried with exponential backoff via Oban; persistent failures escalate to Wazuh alert. |

### 1.4.2 DHIS2 Integration

| Mechanism | Details |
| :--- | :--- |
| **Pull** | `Dhis2SyncWorker` (Oban, daily cron) retrieves waterborne disease indicators from DHIS2 using the DHIS2 Web API (`/api/dataValueSets`). |
| **Push** | SDG6-related WASH indicators computed nightly by `ReportGeneratorWorker` are pushed to the relevant DHIS2 organisation units via authenticated POST. |
| **Mapping layer** | A Gleam-validated mapping table maintains DRC WASH indicator codes to DHIS2 data element UIDs. |
| **Authentication** | OAuth2 bearer tokens for DHIS2; API keys for mWater. Secrets managed in environment-level vault (not committed to source). |

### 1.4.3 External API Standards

All external integration endpoints conform to:
- **REST/JSON** with OpenAPI 3.0 specifications published at `/api/docs`
- **HTTPS only** (TLS 1.2 minimum, TLS 1.3 preferred)
- **API versioning** (`/api/v1/`) to allow non-breaking future extension
- **Rate limiting** via ETS token bucket (per-key and per-IP)

---

## 1.5 Connectivity: Full Offline Mode with Automatic Synchronization

### 1.5.1 Web Portal Offline Capability

The Flutter web portal implements **Service Worker-based offline caching** using the following strategy:

- Static assets (JS, CSS, fonts, map tile cache) cached on first load via PWA manifest
- Last-fetched dashboard data stored in browser IndexedDB for display when offline
- Read-only offline mode: data visible but mutations queued for sync-on-reconnect
- WebSocket reconnection with automatic Phoenix Channel re-subscription on network restoration

### 1.5.2 Mobile App — Full Offline-First Architecture

The mobile app is designed to function **completely offline**; connectivity is treated as an optimization, not a requirement.

**Synchronization Stack:**
```
PostgreSQL WAL (logical replication)
        ↓
  PowerSync Service
        ↓
  Device SQLite (via PowerSync client SDK)
        ↓
  Drift ORM watchers
        ↓
  Riverpod AsyncNotifier (reactive UI)
```

**Key properties:**
- Devices maintain a local working dataset scoped to their province and role
- Replication resumes from the last acknowledged WAL position — only deltas move over the network
- The UI reacts to local SQLite changes; the user is never blocked waiting for a server response
- Offline-collected forms are queued locally and uploaded in batch when connectivity returns via `MobileUploadWorker` (Oban)
- Minimum viable bandwidth: **128 kbps** (enforced design constraint across all payload sizing decisions)

### 1.5.3 Conflict Resolution and Data Integrity

| Scenario | Handling |
| :--- | :--- |
| Concurrent edits to same water point (field agent + office user) | Server-side actor (GenServer) serializes writes; last-write-wins with timestamp; both versions logged |
| Mobile submission of record already updated server-side | PowerSync delta protocol handles; Gleam validator flags conflicts; operator notified |
| Long offline period (>7 days) | Full re-sync triggered; province-scoped Sync Rules limit data volume |

---

## 1.6 Data Sovereignty and Local Hosting

### 1.6.1 Primary Hosting — DRC

The primary SGI-EHA platform is **hosted entirely within the Democratic Republic of the Congo**, in compliance with DRC Law on Electronic Communications and the WASH sector data sovereignty requirements.

**Infrastructure Model:**

| Component | Technology | Hosting |
| :--- | :--- | :--- |
| Bare-metal provisioning | Ubuntu MAAS | DRC data center |
| VM / container lifecycle | MicroCloud + LXD | DRC data center |
| Application nodes (×3) | BEAM cluster on LXD instances | DRC data center |
| Database primary | PostgreSQL 16 + PostGIS | DRC data center |
| GIS services | Martin + pg_featureserv on LXD | DRC data center |
| Edge / security | Nginx + ModSecurity + Suricata | DRC data center |
| Monitoring | Prometheus + Grafana + Wazuh | DRC data center |

<!-- ⚠️ REPLACE: Insert the specific DRC data center name, provider, and certifications (e.g., Tier II/III rating, physical security standards) that will be used for hosting -->
**Proposed DRC Data Center:** [DATA CENTER NAME — TO BE CONFIRMED]
Address: [Physical address in DRC]
Tier Rating: [Tier II / Tier III — confirm and replace]

### 1.6.2 Disaster Recovery — Encrypted Private Cloud Replication

A secondary **encrypted private cloud replication** environment provides disaster recovery and business continuity without relocating operational control outside DRC.

| DR Element | Implementation |
| :--- | :--- |
| WAL streaming | Continuous logical replication to standby replica |
| Nightly encrypted backups | `pgBackRest` with AES-256 encryption |
| Cold storage snapshots | Weekly full-database snapshot retained for 90 days |
| Restore testing | Monthly automated restore test to isolated environment (part of operational runbook) |
| Recovery Time Objective (RTO) | < 4 hours for full platform recovery |
| Recovery Point Objective (RPO) | < 15 minutes (WAL streaming lag) |

<!-- ⚠️ REPLACE: Confirm the private cloud DR provider and its geographic location. Ensure it does not violate DRC data residency requirements for sensitive public-sector data -->

---

## 1.7 GIS Architecture

### 1.7.1 Native PostGIS Integration and OGC Standards

The GIS stack is built natively on **PostGIS 3**, with full compliance to OGC standards:

| Standard | Implementation |
| :--- | :--- |
| **WMS / WMTS** | Martin serving Mapbox Vector Tiles (MVT) over WMTS/XYZ |
| **WFS / OGC API - Features** | pg_featureserv exposing GeoJSON feature APIs |
| **ISO 19115 Metadata** | Stored as JSONB in PostgreSQL; exposed via dedicated Phoenix controller |

### 1.7.2 Why Martin + pg_featureserv Instead of GeoServer

This is a deliberate, performance-first architectural decision:

| Criterion | GeoServer | Martin + pg_featureserv |
| :--- | :--- | :--- |
| Memory footprint | High (JVM) | Very low (Rust binaries) |
| Startup time | Slow | Instantaneous |
| Map payload size | Large raster WMS | Compact MVT (client-side rendering) |
| 128 kbps suitability | Poor | Excellent |
| Operational complexity | High | Minimal |
| PostGIS integration | Good | Native |

### 1.7.3 GIS Access Control

GIS endpoints are protected via Nginx `auth_request` pattern:
1. User authenticates with Phoenix (JWT + session).
2. Tile/feature request hits Nginx.
3. Nginx performs a subrequest to Phoenix `/auth/check` validating JWT and RBAC role.
4. Phoenix returns 200 (allow) or 403 (deny).
5. Martin or pg_featureserv serves the response — both remain stateless and role-unaware.

### 1.7.4 GeoServer / Mapbox Integration Compatibility

The platform's Martin-based tile endpoints are **compatible with GeoServer WMS/WMTS clients and Mapbox GL JS** via standard XYZ tile URL format:
`/tiles/{layer}/{z}/{x}/{y}.mvt`

The Flutter web portal uses `flutter_map` with `VectorTileLayer` consuming Martin-served MVT tiles. The Flutter mobile app uses offline tile caching for pre-downloaded provincial map areas.

---

## 1.8 Performance Targets and How the Architecture Meets Them

| Target | Strategy |
| :--- | :--- |
| **≤ 800 ms** API response | ETS L1 cache for hot reads; efficient Ecto queries; lightweight BEAM concurrency; no expensive in-path aggregations |
| **≤ 1.2 s** web module load | Flutter deferred loading; route-level lazy loading; compressed static assets; CDN-ready asset serving via Nginx |
| **500 concurrent users** | BEAM process model (lightweight per-request process); Phoenix Channels for WebSocket; shared-nothing concurrency; 3-node BEAM cluster |
| **99.5% monthly availability** | OTP supervision (auto-restart); Nginx health checks; 3-node cluster with `libcluster`; durable PostgreSQL session store (no distributed-memory split-brain) |
| **128 kbps tolerance** | MVT vector tiles; PowerSync delta sync; WebSocket diffs only; compact JSON payloads; province-scoped sync rules limit mobile data volume |

---

---

# CHAPTER 2 — CYBERSECURITY STRATEGY & PLAN {#chapter-2}
## Evaluation Weight: 15 Points (15%)

---

## 2.1 Security Architecture Overview

The SGI-EHA cybersecurity posture is **defense-in-depth**: security is not delegated to a single tool or boundary, but layered across network, application, data, identity, and operational levels. The framework aligns with **ISO 27001** principles and adopts a **Zero Trust** model where no internal or external actor is implicitly trusted.

```
┌─────────────────────────────────────────────────────────────┐
│  LAYER 1 — NETWORK EDGE                                      │
│  Nginx TLS termination + ModSecurity WAF (OWASP CRS)         │
│  Suricata IDS/IPS with ML-enhanced anomaly detection         │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  LAYER 2 — APPLICATION                                        │
│  Phoenix auth plugs + RBAC enforcement                        │
│  MFA (TOTP) with configurable code expiry                     │
│  JWT + server-side session validation                         │
│  Rate limiting (ETS token bucket, per-user and per-IP)        │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  LAYER 3 — DATA                                               │
│  PostgreSQL row-level security + encrypted columns            │
│  AES-256 encryption at rest + TLS in transit                  │
│  Audit log tables (immutable append-only event stream)        │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  LAYER 4 — DETECTION & RESPONSE                               │
│  Wazuh SIEM (centralized log aggregation + alerting)          │
│  AI-enhanced Suricata IDS (ML anomaly classification)         │
│  AI Recovery Assistant (log analysis + guided restoration)    │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│  LAYER 5 — OPERATIONAL SECURITY (DevSecOps)                   │
│  CI/CD pipeline with SAST/DAST scans                          │
│  Dependency vulnerability auditing (mix audit / mix deps)     │
│  Infrastructure-as-code security review                       │
│  Quarterly penetration testing                                │
└─────────────────────────────────────────────────────────────┘
```

---

## 2.2 AI-Enhanced Web Application Firewall (WAF)

### 2.2.1 ModSecurity + OWASP Core Rule Set

The platform deploys **ModSecurity** integrated with Nginx at the edge layer, configured with:

- **OWASP Core Rule Set (CRS) v4**: protection against SQLi, XSS, CSRF, path traversal, RFI/LFI, and the OWASP Top 10
- **Paranoia Level 2** configuration balancing strong protection with operational false-positive management
- **DRC-specific rules**: additional WAF rules targeting API abuse patterns specific to mWater/DHIS2 integration endpoints

### 2.2.2 Machine Learning Enhancement for WAF

The WAF is enhanced with an **ML-based behavioral analytics layer** that operates alongside static rules:

| ML Component | Function |
| :--- | :--- |
| **Request anomaly scoring** | Trained on 90 days of baseline traffic; flags statistically anomalous request patterns (unusual parameter lengths, encoding, user-agent fingerprints) |
| **API abuse detection** | Identifies credential stuffing, automated scraping, and volumetric API attacks with behavioral clustering |
| **Adaptive threshold tuning** | False-positive feedback loop allows the ML model to tighten or relax thresholds per endpoint over time |
| **Training infrastructure** | Model retrained weekly on new traffic samples; Wazuh feeds anomaly events into the training pipeline |

<!-- ⚠️ REPLACE: Specify the ML WAF product or framework you will use (e.g., AWS WAF with managed ML rules, a commercial ML WAF product, or a custom-built solution using scikit-learn + Nginx integration). Include version numbers and licensing details -->

---

## 2.3 AI-Enhanced Intrusion Detection System (IDS)

### 2.3.1 Suricata IDS with Machine Learning

**Suricata** is deployed in **IDS/IPS mode** with ML-enhanced detection capabilities:

| Feature | Implementation |
| :--- | :--- |
| **Signature-based detection** | Suricata Emerging Threats ruleset (updated daily) covering known CVEs and attack patterns |
| **ML anomaly detection** | Network traffic behavioral profiling; alerts on statistical deviations from established baselines for each service endpoint |
| **Protocol anomaly detection** | Deep packet inspection for HTTP/HTTPS, DNS, and PostgreSQL wire protocol anomalies |
| **East-west traffic monitoring** | Internal traffic between BEAM nodes, PostgreSQL, and GIS services monitored for lateral movement indicators |
| **Alert enrichment** | Suricata alerts enriched with geolocation, ASN, threat intelligence feeds before forwarding to Wazuh |

### 2.3.2 Wazuh SIEM — Centralized Security Intelligence

**Wazuh** serves as the central SIEM and security operations platform:

- Ingests logs from Nginx, Suricata, Phoenix application logs, PostgreSQL audit logs, OS-level syslog, and SSH sessions
- **MITRE ATT&CK** framework mapping for detected techniques
- Real-time alerting via email, SMS, and in-platform notification (Alerts domain module)
- Dashboards for security operations with drill-down to raw events
- Compliance reporting aligned with ISO 27001 audit evidence requirements

---

## 2.4 AI Recovery Assistant

### 2.4.1 Architecture and Capabilities

The **AI Recovery Assistant** is an integrated module within the SGI-EHA platform providing:

| Capability | Description |
| :--- | :--- |
| **Automated log analysis** | On-demand and scheduled analysis of Wazuh, Nginx, PostgreSQL, and Phoenix logs to identify root cause of incidents |
| **Sinister diagnostics** | Guided incident investigation workflow: timeline reconstruction, affected data scope identification, blast radius assessment |
| **Guided data restoration** | Step-by-step recovery procedure generation based on incident type, affected modules, and available backup points |
| **Natural language interface** | Operators can query the assistant in French or English: "Qu'est-ce qui s'est passé entre 14h00 et 15h30 hier?" |
| **Forensic timeline reconstruction** | Cross-correlates events from multiple log sources to reconstruct attack or failure timelines |

### 2.4.2 Technical Implementation

<!-- ⚠️ REPLACE: Specify the LLM/AI backend you will use for the Recovery Assistant (e.g., a self-hosted model like Llama 3 for data sovereignty, an API-based service, or a commercial tool). Justify the choice against DRC data sovereignty requirements — avoid sending sensitive log data to foreign cloud AI APIs -->

The AI Recovery Assistant is implemented as a dedicated Oban worker pipeline with:

1. **Log ingestion**: structured log events from Wazuh API streamed into a local vector database
2. **Contextual retrieval (RAG)**: incident queries retrieve relevant log excerpts, system documentation, and past incident reports
3. **Inference**: a locally-hosted LLM generates diagnostic narratives and recovery guidance
4. **Output**: recommendations surfaced in the SGI-EHA web portal for authorized security administrators

**Sovereignty note:** All AI inference runs on DRC-hosted infrastructure. No log data or system events are transmitted to foreign API endpoints.

---

## 2.5 Multi-Factor Authentication (MFA)

### 2.5.1 MFA Architecture

All SGI-EHA users with privileged or data-write access are required to enroll in **MFA using TOTP (Time-based One-Time Password)**:

| MFA Component | Implementation |
| :--- | :--- |
| **Protocol** | RFC 6238 TOTP (compatible with Google Authenticator, Authy, and hardware tokens) |
| **Code length** | 6-digit TOTP code |
| **Configurable expiry** | Default 30-second window; configurable per user role (30s / 60s / 90s) via admin panel |
| **Backup codes** | 10 single-use backup codes generated at enrollment; encrypted at rest |
| **SMS fallback** | Optional SMS OTP for users without smartphone access; subject to network availability |
| **Hardware token support** | FIDO2/WebAuthn support for high-privilege accounts (system administrators, Cybersecurity Specialist) |

### 2.5.2 MFA Enforcement Policy

| Role | MFA Requirement |
| :--- | :--- |
| System Administrator | Mandatory (TOTP + FIDO2 recommended) |
| Province Director | Mandatory (TOTP) |
| Data Entry Agent | Mandatory (TOTP or SMS) |
| Read-Only Analyst | Required for access to PII or confidential financial data |
| Public citizen (Alert module) | Not required |

### 2.5.3 Session Security Controls

- JWT tokens issued post-MFA with configurable TTL (default: 8 hours for web, 24 hours for mobile)
- Concurrent session limit per user (configurable, default: 3 active sessions)
- Server-side session revocation via PostgreSQL `user_sessions` table — immediate effect across all nodes
- Login anomaly detection: geolocation jump, unusual hours, new device fingerprint → automatic MFA re-challenge

---

## 2.6 Audit Logs and Event Traceability

### 2.6.1 Immutable Audit Log Architecture

All user and system actions across every SGI-EHA module generate **immutable audit log entries** stored in a dedicated PostgreSQL schema (`audit.*`):

```sql
-- Example audit log entry structure
audit.events (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  occurred_at     TIMESTAMPTZ NOT NULL DEFAULT now(),
  actor_id        UUID REFERENCES users(id),
  actor_ip        INET,
  actor_session   UUID,
  module          TEXT NOT NULL,        -- e.g., 'water', 'finance', 'admin'
  action          TEXT NOT NULL,        -- e.g., 'CREATE', 'UPDATE', 'DELETE', 'LOGIN', 'EXPORT'
  resource_type   TEXT,                 -- e.g., 'water_point', 'user', 'report'
  resource_id     UUID,
  before_state    JSONB,
  after_state     JSONB,
  metadata        JSONB                 -- request_id, user_agent, etc.
)
```

**Immutability enforcement:**
- Audit table has **no UPDATE or DELETE permissions** granted to the application role
- Separate PostgreSQL role with INSERT-only access used by the audit writer
- Monthly audit log hash chains generated and stored out-of-band for tamper evidence

### 2.6.2 Advanced Event Reconstruction Capability

The platform provides **forensic event reconstruction** across all modules:

| Capability | Implementation |
| :--- | :--- |
| **Cross-module timeline** | Query `audit.events` across any module combination for a given actor, resource, or time window |
| **Before/After state diffing** | `before_state` and `after_state` JSONB columns enable field-level change tracking |
| **Session replay** | Reconstruct the complete sequence of actions taken within a user session |
| **Bulk operation audit** | CSV export operations, bulk updates, and API calls audited with record counts and payload hashes |
| **Security event correlation** | Audit events linked to Wazuh security events via `request_id` for cross-layer incident investigation |

### 2.6.3 Audit Log Retention and Access

| Retention Period | Storage | Access |
| :--- | :--- | :--- |
| Hot (0–12 months) | PostgreSQL `audit.events` (indexed) | Security admin, via audit portal |
| Warm (12–36 months) | Compressed PostgreSQL partition | Security admin, via export request |
| Cold (36+ months) | Encrypted archive (pgBackRest snapshot) | Formal request process, restored on demand |

---

## 2.7 DevSecOps and Compliance Framework

### 2.7.1 DevSecOps Pipeline

Security is embedded in every stage of the development and deployment lifecycle:

| Phase | Security Activity |
| :--- | :--- |
| **Code** | Gleam's type system eliminates entire classes of runtime vulnerabilities; Elixir/Erlang memory safety |
| **Build** | `mix audit` for dependency CVE scanning; automated checks in CI pipeline |
| **Test** | Automated DAST scanning against staging environment; OWASP ZAP integration |
| **Deploy** | Infrastructure-as-code reviewed before promotion; no manual production changes |
| **Operate** | Wazuh + Suricata continuous monitoring; monthly vulnerability scans |
| **Respond** | AI Recovery Assistant; documented incident response runbooks; quarterly drills |

### 2.7.2 ISO 27001 Alignment

| ISO 27001 Control Domain | SGI-EHA Implementation |
| :--- | :--- |
| A.9 Access Control | RBAC in Phoenix plugs; MFA; session management |
| A.10 Cryptography | TLS 1.3; AES-256 at rest; pgBackRest encrypted backups |
| A.12 Operations Security | Oban background jobs; Wazuh monitoring; change management |
| A.13 Communications Security | Nginx TLS; WAF; Suricata IDS; encrypted API channels |
| A.16 Incident Management | AI Recovery Assistant; documented runbooks; quarterly drills |
| A.17 Business Continuity | DR replication; pgBackRest; RTO < 4h; RPO < 15min |

### 2.7.3 Zero Trust Implementation

| Zero Trust Principle | Implementation |
| :--- | :--- |
| **Verify explicitly** | Every request validated: JWT + session + RBAC + MFA state |
| **Least privilege** | PostgreSQL roles scoped to minimum necessary; RBAC roles scoped to province and function |
| **Assume breach** | Internal traffic monitored; East-West Suricata rules; audit logs on all mutations |

---

---

# CHAPTER 3 — PERSONNEL COMPETENCIES {#chapter-3}
## Evaluation Weight: 20 Points (20%)

---

## 3.1 Team Overview

The proposed team comprises seven key experts meeting all minimum requirements defined in the DDP. All experts are available for the full duration of the project and have signed the attached Environmental and Social Code of Conduct (Annex A).

<!-- ⚠️ REPLACE: Confirm actual availability and update the table with real names. If using sub-consultants, identify them and provide their firm details -->

| Position | Proposed Expert | Total Exp. | Similar Exp. | Availability |
| :--- | :--- | :--- | :--- | :--- |
| Chef de Mission | [See Section 3.2] | 12 years | 7 years | 100% |
| Software Architect | [See Section 3.3] | 10 years | 5 years | 100% |
| Back-end Developer | [See Section 3.4] | 6 years | 6 years | 100% |
| Front-end Developer | [See Section 3.5] | 5 years | 5 years | 100% |
| Data Analyst | [See Section 3.6] | 8 years | 8 years | 100% |
| Cybersecurity Specialist | [See Section 3.7] | 11 years | 6 years | 100% |
| Support Technician | [See Section 3.8] | 5 years | 5 years | 100% |

---

## 3.2 Chef de Mission

<!-- ⚠️ REPLACE: All personal details below are illustrative and must be replaced with the actual expert's verified information -->

**Name:** [Jean-Baptiste Mwangi Kalonji] — ⚠️ REPLACE WITH ACTUAL NAME
**Nationality:** [DRC / [Country]] — ⚠️ REPLACE
**Total Experience:** 12 years
**Similar Project Experience:** 7 years

**Professional Summary:**
[Jean-Baptiste] has led the design and delivery of five national-scale information management systems across Sub-Saharan Africa, with a specific focus on public-sector WASH and infrastructure data platforms. He has coordinated multi-donor technical assistance projects in DRC, Rwanda, and Côte d'Ivoire, managing teams of 15+ technical specialists.

**Key Competencies:**
- SGI project coordination in Africa (5 SGI deployments, 3 in DRC context)
- GIS mastery: ArcGIS, QGIS, PostGIS; spatial analysis for WASH infrastructure mapping
- Multi-stakeholder coordination (World Bank, AFD, UNICEF, government counterparts)
- French and English fluency; Swahili and Lingala working knowledge
- PMP certified

**Relevant Experience (minimum 2 SGI-type projects in Africa):**
<!-- ⚠️ REPLACE: Insert actual project references. The two examples below are illustrative and must be replaced -->

| # | Project | Client | Country | Period | Role |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | [National WASH MIS Design and Deployment] | [Ministry of Water / World Bank] | [DRC / Rwanda] | [2021–2023] | Project Director |
| 2 | [GIS-Based Sanitation Tracking Platform] | [UNICEF / AfDB] | [Côte d'Ivoire] | [2019–2021] | Team Leader |

**Detailed CV:** See Annex C — CV-01

---

## 3.3 Software Architect

<!-- ⚠️ REPLACE: Replace with actual expert details -->

**Name:** [Marie-Claire Nkosi Banza] — ⚠️ REPLACE WITH ACTUAL NAME
**Nationality:** [DRC / [Country]] — ⚠️ REPLACE
**Total Experience:** 10 years
**Similar Project Experience:** 5 years

**Professional Summary:**
[Marie-Claire] is a senior software architect with deep expertise in distributed systems design for bandwidth-constrained environments. She holds certifications in cloud architecture and has designed API-first platforms for three African public-sector clients.

**Key Competencies:**
- **MERISE** methodology: Entity-Relationship modeling, data flow diagrams for DRC government system design
- **C4 Model**: system context, container, component, and code-level architecture documentation
- **REST API design**: OpenAPI 3.0, versioning strategies, API gateway patterns
- **GIS Integration**: GeoServer, Mapbox, PostGIS, OGC standards (WMS/WFS/WMTS)
- **Elixir/Phoenix**, **Node.js**, **PostgreSQL**; experience with BEAM supervision models
- Fluent in French and English

**MERISE and C4 Model Deliverables Planned:**
- Full MERISE Conceptual Data Model (MCD) and Physical Data Model (MPD) for all 6 domain modules
- C4 Model documentation: System Context, Container, and Component diagrams for all layers
- API specification in OpenAPI 3.0 format for all internal and external interfaces
- GIS layer metadata schema aligned with ISO 19115

**Detailed CV:** See Annex C — CV-02

---

## 3.4 Back-end Developer

<!-- ⚠️ REPLACE: Replace with actual expert details -->

**Name:** [Emmanuel Tshibangu Ntumba] — ⚠️ REPLACE WITH ACTUAL NAME
**Nationality:** [DRC / [Country]] — ⚠️ REPLACE
**Total Experience:** 6 years
**Similar Project Experience:** 6 years

**Professional Summary:**
[Emmanuel] is a back-end specialist with production experience in high-availability public-sector platforms. He has built REST APIs serving 10,000+ daily active users across East Africa, with expertise in database performance tuning and offline synchronization architectures.

**Key Competencies:**
- **Python** (FastAPI, Django REST Framework): 4 years production experience
- **Elixir/Phoenix**: 3 years; OTP supervision, GenServer, Phoenix Channels
- **Node.js** (Express, NestJS): 5 years
- **PostgreSQL** (advanced): query optimization, partitioning, PostGIS, WAL replication
- **MongoDB**: document modeling, aggregation pipelines
- **PowerSync** offline sync architecture
- **Oban** background job patterns

**Detailed CV:** See Annex C — CV-03

---

## 3.5 Front-end Developer

<!-- ⚠️ REPLACE: Replace with actual expert details -->

**Name:** [Patience Lukusa Kabila] — ⚠️ REPLACE WITH ACTUAL NAME
**Nationality:** [DRC / [Country]] — ⚠️ REPLACE
**Total Experience:** 5 years
**Similar Project Experience:** 5 years

**Professional Summary:**
[Patience] specializes in offline-first mobile and web applications for field data collection in low-connectivity environments. She has built Flutter applications deployed across three African countries for WASH and health sector data collection programs.

**Key Competencies:**
- **Flutter** (primary): web portal and offline-first mobile app; Riverpod state management; Drift ORM; PowerSync client integration
- **React** (secondary): 3 years; component design, state management (Redux/Zustand)
- **Vue.js**: 2 years
- **Leaflet / Mapbox GL JS**: interactive GIS map widgets; vector tile rendering
- `flutter_map` with `VectorTileLayer` for MVT tile consumption
- Offline PWA architecture: Service Workers, IndexedDB, Web App Manifests
- Responsive design aligned with DRC government graphic charter standards
- French and English fluency

**Detailed CV:** See Annex C — CV-04

---

## 3.6 Data Analyst

<!-- ⚠️ REPLACE: Replace with actual expert details -->

**Name:** [Pascal Ilunga Wa Ilunga] — ⚠️ REPLACE WITH ACTUAL NAME
**Nationality:** [DRC / [Country]] — ⚠️ REPLACE
**Total Experience:** 8 years
**Similar Project Experience:** 8 years

**Professional Summary:**
[Pascal] has spent 8 years in WASH sector data analytics, with direct experience producing SDG6 indicator reports for UN agencies and the DRC Ministry of Water. He is DHIS2-certified and has built Power BI dashboards consumed by 200+ ministry users across 4 provinces.

**Key Competencies:**
- **Power BI**: Advanced DAX, custom visuals, paginated reports, RLS-based security; direct integration with PostgreSQL
- **DHIS2**: Data element configuration, organisation unit management, analytics API, custom indicators; DHIS2 Academy certified
- **SDG6 indicators**: Targets 6.1–6.6 indicator methodology; JMP definitions; national reporting alignment
- SQL analytics: complex aggregations, window functions, materialized views
- Statistical analysis: R (tidyverse, ggplot2); Python (pandas, matplotlib)
- French fluency; working knowledge of English

**SDG6 Indicator Coverage:**
The platform will compute and report on the following SDG6 sub-indicators for DRC:

| SDG6 Target | Indicator | Data Source |
| :--- | :--- | :--- |
| 6.1 Safe water access | % population using safely managed drinking water | Water domain + DHIS2 |
| 6.2 Sanitation & hygiene | % using safely managed sanitation | Sanitation domain |
| 6.2 Hygiene | % with basic handwashing facility | Hygiene domain |
| 6.3 Water quality | % wastewater safely treated | Sanitation domain |
| 6.5 Water management | Degree of IWRM implementation | Regulation domain |

**Detailed CV:** See Annex C — CV-05

---

## 3.7 Cybersecurity Specialist

<!-- ⚠️ REPLACE: Replace with actual expert details -->

**Name:** [Aristide Nzongola-Ntalaja] — ⚠️ REPLACE WITH ACTUAL NAME
**Nationality:** [DRC / [Country]] — ⚠️ REPLACE
**Total Experience:** 11 years
**Similar Project Experience:** 6 years

**Professional Summary:**
[Aristide] is a senior cybersecurity specialist with ISO 27001 Lead Implementer certification and extensive experience securing public-sector digital platforms in Africa. He has designed and operated security operations centers (SOC) for three government agencies, and led two successful ISO 27001 certification audits.

**Key Competencies:**
- **ISO 27001**: Lead Implementer certified; 2 successful ISMS implementations in government context
- **DevSecOps**: CI/CD pipeline security integration; SAST (Semgrep); DAST (OWASP ZAP); container security scanning
- **WAF**: ModSecurity + OWASP CRS configuration and tuning; ML-based WAF behavioral analytics
- **Zero Trust Architecture**: policy design, micro-segmentation, continuous verification
- **MFA**: TOTP/FIDO2 deployment; enterprise MFA rollout for 500+ users
- **Wazuh SIEM** and **Suricata IDS/IPS**: deployment, rule development, alert tuning
- **pgBackRest** disaster recovery; restore procedure development and testing
- French and English fluency

**Detailed CV:** See Annex C — CV-06

---

## 3.8 Support Technician (GIS Server Administrator)

<!-- ⚠️ REPLACE: Replace with actual expert details -->

**Name:** [Cécile Mbuyi Tshiela] — ⚠️ REPLACE WITH ACTUAL NAME
**Nationality:** [DRC / [Country]] — ⚠️ REPLACE
**Total Experience:** 5 years
**Similar Project Experience:** 5 years

**Professional Summary:**
[Cécile] is a Linux system administrator and GIS server specialist with hands-on experience managing PostGIS databases and tile servers for WASH sector clients in DRC. She has administered Ubuntu MAAS and LXD environments and maintained GIS infrastructure serving provincial WASH teams.

**Key Competencies:**
- **Linux Administration**: Ubuntu Server; systemd; LXD/MicroCloud; Ubuntu MAAS bare-metal provisioning
- **PostGIS**: schema design, spatial index optimization, raster/vector data management, VACUUM tuning for geospatial workloads
- **GIS Server Management**: Martin (MVT tile server); pg_featureserv; Nginx configuration for GIS endpoints
- **PostgreSQL DBA**: WAL management, replication monitoring, pgBackRest backup/restore operations
- **Monitoring**: Prometheus + Grafana dashboards for infrastructure metrics; Wazuh agent deployment
- French fluency

**Detailed CV:** See Annex C — CV-07

---

---

# CHAPTER 4 — IMPLEMENTATION STRATEGY {#chapter-4}
## Evaluation Weight: 15 Points (15%)

---

## 4.1 Delivery Guarantee: Full Platform in Less Than 7 Months

The implementation plan delivers **full, simultaneous platform launch across the central office and all four target provinces within 6 months and 3 weeks** — within the 7-month maximum. A structured 100-day accompaniment plan follows over the subsequent year.

### 4.1.1 Master Implementation Schedule

| Phase | Duration | Weeks | Key Deliverables |
| :--- | :--- | :--- | :--- |
| **Phase 0** — Inception & Environment Setup | 2 weeks | W1–W2 | Kickoff, environment provisioning, team onboarding |
| **Phase 1** — Foundation & Architecture | 3 weeks | W3–W5 | Monorepo scaffold, OTP supervision, database, Nginx, auth |
| **Phase 2** — Core Domain Modules | 4 weeks | W6–W9 | Water, Sanitation, Projects, Regulation, Finance, Alerts |
| **Phase 3** — Real-Time & Background Jobs | 2 weeks | W10–W11 | PubSub/Channels, Oban workers, mWater + DHIS2 integration |
| **Phase 4** — GIS Integration | 2 weeks | W12–W13 | Martin, pg_featureserv, PostGIS layers, Nginx auth_request |
| **Phase 5** — Offline Sync (PowerSync) | 2 weeks | W14–W15 | PowerSync deployment, WAL rules, mobile SQLite wiring |
| **Phase 6** — Web Portal (Flutter) | 3 weeks | W16–W18 | Dashboards, admin, maps, RBAC management, SDG reporting |
| **Phase 7** — Mobile App (Flutter) | 3 weeks | W16–W18 | Offline forms, sync, map tiles, field workflows (parallel with Phase 6) |
| **Phase 8** — Security & Observability | 2 weeks | W19–W20 | WAF hardening, Wazuh/Suricata, AI assistant, Prometheus/Grafana |
| **Phase 9** — Provincial Deployment | 2 weeks | W21–W22 | Simultaneous deployment: Central + 4 provinces |
| **Phase 10** — UAT & Performance Validation | 2 weeks | W23–W24 | User acceptance testing, load testing, performance benchmark |
| **Phase 11** — Go-Live + Handover | 1 week | W25–W27 | Production launch, documentation handover, team training |
| **Total** | **≤ 27 weeks** | | **Full delivery ≤ 7 months** ✓ |

### 4.1.2 Critical Path and Risk Mitigation

| Critical Path Item | Risk | Mitigation |
| :--- | :--- | :--- |
| DRC data center provisioning | Physical infrastructure delays | Begin parallel provisioning in Phase 0; confirm hosting provider by Day 1 |
| DHIS2/mWater API access credentials | Dependency on third-party systems | Request API credentials at project kickoff; implement mock adapters for parallel dev |
| Provincial connectivity for deployment | Unstable links during deployment | Offline-first deployment package; local deployment agent in each province |
| User acceptance from ministry stakeholders | Change resistance | Embed provincial champions early; prototype demos at Week 10 |

---

## 4.2 Simultaneous Deployment: Central Office and All Four Provinces

### 4.2.1 Deployment Architecture by Location

<!-- ⚠️ REPLACE: Confirm the four target provinces from the DDP. The provinces below are illustrative based on common DRC WASH project scope -->

| Location | Role | Infrastructure | Connectivity Mode |
| :--- | :--- | :--- | :--- |
| **Kinshasa** (Central) | Primary server, BEAM cluster, PostgreSQL primary, all services | Full on-premise DRC data center | Primary (direct) |
| **[Province 1]** | Provincial web portal access; mobile sync | Thin client + internet access to central | VPN over internet |
| **[Province 2]** | Provincial web portal access; mobile sync | Thin client + internet access to central | VPN over internet |
| **[Province 3]** | Provincial web portal access; mobile sync | Thin client + internet access to central | VPN over internet |
| **[Province 4]** | Provincial web portal access; mobile sync | Thin client + internet access to central | VPN over internet |

<!-- ⚠️ REPLACE: Insert the actual four province names from the DDP project scope -->

### 4.2.2 Simultaneous Launch Guarantee

The simultaneous launch is achieved through:

1. **Centralized platform with provincial VPN connectivity**: All application logic runs on the Kinshasa-based cluster; provinces connect via secure VPN tunnels. No provincial servers required, eliminating provincial infrastructure deployment risk.
2. **Pre-loaded provincial data packages**: Province-scoped datasets, organization units, user accounts, and Sync Rules are prepared and tested two weeks before go-live.
3. **Parallel training execution**: Training sessions conducted simultaneously in all five locations during Weeks 24–25 via a combination of in-person trainers and video-conference facilitation.
4. **Offline deployment packages**: Provincial field agents receive pre-configured mobile devices with offline map tiles and initial data synchronized before traveling to areas with no connectivity.
5. **Go-live ceremony**: A coordinated simultaneous launch event at Week 27, with video link between all provincial teams and Kinshasa central office.

---

## 4.3 Training Plan

### 4.3.1 Training Program Overview

The training program is structured in four components and delivered to all user groups before and after go-live:

| Training Module | Target Audience | Duration | Format |
| :--- | :--- | :--- | :--- |
| **T1 — System Administrator Training** | IT staff (central + provincial) | 3 days | In-person workshop |
| **T2 — Platform User Training** | Ministry staff, provincial directors, data entry agents | 2 days per location (5 locations) | In-person + video |
| **T3 — GIS and Reporting Training** | Data analysts, GIS officers | 2 days | In-person workshop |
| **T4 — Cybersecurity and Incident Response** | Security team, IT managers | 2 days | In-person workshop |

### 4.3.2 Disaster Simulation and Post-Sinister Recovery Training

The Cybersecurity training (T4) includes **mandatory practical disaster simulation exercises**:

| Exercise | Description | Duration |
| :--- | :--- | :--- |
| **Simulated Data Breach** | Controlled exercise: security team detects and responds to a simulated API intrusion; uses Wazuh SIEM to trace, contain, and document the incident | 4 hours |
| **Database Corruption Recovery** | Simulated PostgreSQL corruption event; team practices point-in-time recovery using pgBackRest; validates RPO/RTO objectives | 3 hours |
| **Ransomware Tabletop Exercise** | Tabletop scenario: ransomware detection, system isolation, backup-based restoration, communication protocol | 2 hours |
| **Full Platform Recovery Drill** | Complete platform shutdown + full restoration from DR backup; validates 4-hour RTO target | 4 hours |

### 4.3.3 Internal API Connection Management Training

A dedicated 3-hour workshop covers **management of internal API connections**, including:

- Overview of the REST/JSON API architecture and the nucleus model
- How to monitor API health via Prometheus/Grafana dashboards
- How to configure and test mWater and DHIS2 synchronization jobs
- How to read and interpret API error logs in Wazuh
- How to trigger manual sync operations via the admin portal
- API key rotation procedures for mWater and DHIS2 credentials

---

## 4.4 100-Day Accompaniment Plan Over One Year

Post go-live, the team provides a **structured 100-day accompaniment program spread over 12 months**, including quarterly performance audits.

### 4.4.1 Accompaniment Timeline

| Period | Months | Accompaniment Level | Activities |
| :--- | :--- | :--- | :--- |
| **Intensive Phase** | Months 1–2 (Days 1–60) | 40 days | Daily check-ins; incident response within 4 hours; weekly performance review; immediate bug resolution |
| **Stabilization Phase** | Months 3–4 (Days 61–100) | 30 days | Bi-weekly check-ins; incident response within 8 hours; first quarterly audit; training reinforcement for identified gaps |
| **Autonomous Support** | Months 5–9 | 20 days | Monthly check-ins; remote support; second and third quarterly audits; knowledge transfer completion |
| **Annual Review** | Months 10–12 | 10 days | Fourth quarterly audit; annual performance report; system optimization recommendations; renewal planning |
| **Total** | **12 months** | **100 days** | ✓ |

### 4.4.2 Quarterly Performance Audits

Four quarterly audits are conducted at Months 3, 6, 9, and 12, each covering:

| Audit Area | Metrics Reviewed |
| :--- | :--- |
| **Availability** | Monthly uptime vs. 99.5% SLA target |
| **Performance** | API response time (≤ 800 ms); module load time (≤ 1.2 s); concurrent user load |
| **Security** | Wazuh alert trends; WAF block rates; MFA adoption rate by user group; vulnerability scan findings |
| **Data Quality** | Sync success rates (mWater, DHIS2); offline/online reconciliation error rates; audit log completeness |
| **User Adoption** | Active user counts by province; mobile app sync frequency; reporting module usage |
| **Capacity** | Database growth rate; storage headroom; recommendations for hardware scaling |

Each audit produces a written report delivered to CEP-O within 5 business days of the audit date.

### 4.4.3 Support Commitment

| Support Level | Response Time | Channel |
| :--- | :--- | :--- |
| Critical (platform down) | < 2 hours | Phone + email |
| High (major feature unavailable) | < 4 hours | Email + ticketing system |
| Medium (partial feature impacted) | < 8 hours | Ticketing system |
| Low (cosmetic / enhancement request) | < 3 business days | Ticketing system |

---

---

# ANNEX A — ADMINISTRATIVE DOCUMENTS {#annex-a}

<!-- ⚠️ REPLACE: The following section lists required administrative documents. Each must be physically attached to the proposal as a separate document. Placeholders indicate what must be provided -->

## A.1 Bid Guarantee

**Required:** Original bank guarantee of **USD 20,000**, valid until **July 2026**.

> **Document status:** ⚠️ REPLACE — Attach original bank guarantee letter from [Name of Bank], dated [Date], issued in favor of CEP-O, for the amount of USD 20,000, valid through [July 31, 2026 or per DDP requirement]. Confirm the exact beneficiary name and wording required by the DDP.

## A.2 Financial Capacity — Proof of Liquid Assets or Credit Lines

**Required:** Proof of liquid assets or credit lines of at least **USD 230,000**.

> **Document status:** ⚠️ REPLACE — Attach one or more of the following: (1) bank statement(s) showing liquid assets ≥ USD 230,000, (2) letter of credit or credit line confirmation from a bank totaling ≥ USD 230,000, (3) audited financial statements for the last 2–3 fiscal years demonstrating sufficient financial health. Ensure documents are dated within 6 months of the submission deadline.

## A.3 Experience — Two Similar Contracts Since 2020

**Required:** Evidence of at least **two similar contracts** since 2020 involving interoperable APIs and offline/online synchronization.

<!-- ⚠️ REPLACE: The two contract references below are illustrative. Replace with actual signed contracts, completion certificates, or client reference letters -->

### Reference Contract 1

| Field | Details |
| :--- | :--- |
| **Contract Title** | [National Health Information System Deployment with mWater/DHIS2 Integration] — ⚠️ REPLACE |
| **Client Name** | [Ministry of Health, Republic of [Country]] — ⚠️ REPLACE |
| **Client Contact** | [Name, Title, Email, Phone] — ⚠️ REPLACE |
| **Contract Value** | [USD XXX,XXX] — ⚠️ REPLACE |
| **Period** | [2021–2023] — ⚠️ REPLACE |
| **Interoperable APIs** | Yes: DHIS2 Push/Pull API, REST/JSON integration with [system name] |
| **Offline/Online Sync** | Yes: Mobile app with SQLite offline mode and automatic background sync |
| **Proof Attached** | Completion certificate / contract extract (Annex B — Reference 1) |

### Reference Contract 2

| Field | Details |
| :--- | :--- |
| **Contract Title** | [WASH GIS Platform with Offline Field Data Collection — Côte d'Ivoire] — ⚠️ REPLACE |
| **Client Name** | [UNICEF / Direction de l'Hydraulique, [Country]] — ⚠️ REPLACE |
| **Client Contact** | [Name, Title, Email, Phone] — ⚠️ REPLACE |
| **Contract Value** | [USD XXX,XXX] — ⚠️ REPLACE |
| **Period** | [2020–2022] — ⚠️ REPLACE |
| **Interoperable APIs** | Yes: mWater API synchronization, OGC WFS/WMS integration |
| **Offline/Online Sync** | Yes: Offline-first mobile data collection with background reconciliation |
| **Proof Attached** | Completion certificate / contract extract (Annex B — Reference 2) |

## A.4 Manufacturer Authorizations

**Required:** Mandatory authorization letters from software editors for any commercial (non-open-source) software used.

> **Document status:** ⚠️ REVIEW — The SGI-EHA architecture is designed primarily on open-source components (Phoenix/Elixir: Apache 2.0; PostgreSQL: PostgreSQL License; Flutter: BSD; Nginx: BSD; Suricata: GPL 2.0; Wazuh: GPL 2.0; Martin: Apache 2.0; pg_featureserv: Apache 2.0). Manufacturer authorization letters are therefore NOT required for core platform components.

> **Exception — PowerSync:** PowerSync is a commercial product. ⚠️ REPLACE — Attach the official authorization letter from PowerSync (or its DRC/Africa distributor), confirming the firm's authorization to deploy and resell PowerSync licenses for this project. Contact: [PowerSync sales contact — check powersync.com for current Africa partner].

> **Exception — ML WAF component (if commercial):** ⚠️ REPLACE — If a commercial ML WAF product is selected (vs. custom-built), attach the corresponding authorization letter.

> **Exception — Power BI:** Power BI is a Microsoft commercial product. ⚠️ REPLACE — If deploying Power BI Premium/Embedded, attach a Microsoft Partner authorization letter or confirm the appropriate licensing model.

## A.5 Environmental and Social Code of Conduct

**Required:** A signed ES Code of Conduct for all proposed personnel.

> **Document status:** ⚠️ REPLACE — Attach signed ES Code of Conduct forms for all seven key experts listed in Chapter 3. Forms must use the template provided in the DDP (or the client's prescribed format). Ensure all forms are dated and wet-signed.

**Persons required to sign:**
1. [Chef de Mission name]
2. [Software Architect name]
3. [Back-end Developer name]
4. [Front-end Developer name]
5. [Data Analyst name]
6. [Cybersecurity Specialist name]
7. [Support Technician name]

## A.6 Litigation History — Form ANT-2

**Required:** Completed Form ANT-2 covering the last 5 years (2021–2026).

> **Document status:** ⚠️ REPLACE — Complete and attach the ANT-2 Litigation History form from the DDP annex, covering all significant legal proceedings, arbitration, or disputes involving the firm over the period 2021–2026. If the firm has no reportable litigation, state this explicitly in the form.

## A.7 Beneficial Ownership Disclosure Form

**Required:** Completed Beneficial Ownership Disclosure Form.

> **Document status:** ⚠️ REPLACE — Complete and attach the Beneficial Ownership Disclosure Form from the DDP annex. This must identify all natural persons who directly or indirectly own or control more than [threshold, typically 10% or 25%] of the submitting firm. Consult the DDP for the specific threshold and form template.

---

# ANNEX B — PAST EXPERIENCE REFERENCES {#annex-b}

<!-- ⚠️ REPLACE: Attach physical copies of completion certificates, contract extracts, or client reference letters for the two contracts listed in Section A.3 -->

**Reference 1:** [Contract title and completion certificate] — ⚠️ REPLACE — Physical document to be attached

**Reference 2:** [Contract title and completion certificate] — ⚠️ REPLACE — Physical document to be attached

---

# ANNEX C — CURRICULUM VITAE OF KEY EXPERTS {#annex-c}

<!-- ⚠️ REPLACE: Attach a complete CV for each of the seven key experts in the format prescribed by the DDP. Each CV should include: personal details, education, professional experience (chronological), list of relevant projects with client names and contact references, and a signed declaration of availability and accuracy -->

- **CV-01:** [Chef de Mission] — ⚠️ REPLACE — Attach complete CV
- **CV-02:** [Software Architect] — ⚠️ REPLACE — Attach complete CV
- **CV-03:** [Back-end Developer] — ⚠️ REPLACE — Attach complete CV
- **CV-04:** [Front-end Developer] — ⚠️ REPLACE — Attach complete CV
- **CV-05:** [Data Analyst] — ⚠️ REPLACE — Attach complete CV
- **CV-06:** [Cybersecurity Specialist] — ⚠️ REPLACE — Attach complete CV
- **CV-07:** [Support Technician] — ⚠️ REPLACE — Attach complete CV

---

# ANNEX D — MANUFACTURER AUTHORIZATIONS {#annex-d}

<!-- ⚠️ REPLACE: Attach authorization letters from software vendors as described in Section A.4. If all software is open-source and no commercial authorizations are needed, include a written statement to that effect, listing the open-source licenses of each component -->

**D.1 — PowerSync Authorization Letter** — ⚠️ REPLACE — Physical letter from PowerSync to be attached

**D.2 — [Commercial ML WAF product, if applicable]** — ⚠️ REPLACE — Physical letter to be attached

**D.3 — Microsoft Power BI Authorization (if applicable)** — ⚠️ REPLACE — Physical letter or licensing confirmation to be attached

**D.4 — Open Source License Statement** (if no commercial software)
The following core platform components are open-source and require no manufacturer authorization:

| Component | License | Version |
| :--- | :--- | :--- |
| Elixir / Phoenix Framework | Apache 2.0 | Phoenix 1.7+ |
| Gleam | Apache 2.0 | 1.x |
| PostgreSQL + PostGIS | PostgreSQL License / GPL 2.0 | PG 16 + PostGIS 3 |
| Flutter | BSD 3-Clause | 3.x |
| Nginx | BSD 2-Clause | 1.25+ |
| ModSecurity | Apache 2.0 | 3.x |
| Suricata | GPL 2.0 | 7.x |
| Wazuh | GPL 2.0 | 4.x |
| Martin | Apache 2.0 | 0.14+ |
| pg_featureserv | Apache 2.0 | 1.3+ |
| Prometheus + Grafana | Apache 2.0 | Current |
| OpenTelemetry | Apache 2.0 | Current |
| pgBackRest | MIT | 2.x |

---

*End of Technical Proposal*

---

> **CHECKLIST BEFORE PRINTING AND SUBMISSION:**
>
> - [ ] All ⚠️ REPLACE markers have been resolved with actual data
> - [ ] Document translated to French
> - [ ] Submission letter printed on company letterhead and wet-signed
> - [ ] All seven CVs attached in DDP-prescribed format
> - [ ] All seven ES Code of Conduct forms attached, signed and dated
> - [ ] Bank guarantee (original, USD 20,000, valid to July 2026) attached
> - [ ] Proof of financial capacity (USD 230,000) attached
> - [ ] Two past reference contracts with completion certificates attached
> - [ ] Form ANT-2 completed and attached
> - [ ] Beneficial Ownership Disclosure Form completed and attached
> - [ ] Manufacturer authorization letters attached for all commercial software
> - [ ] Technical proposal: 1 original + 4 hard copies + 1 USB drive (searchable PDF + Word)
> - [ ] Financial proposal: 1 original + 2 hard copies + 1 USB drive (Excel)
> - [ ] Outer envelope sealed and marked with project reference and "NE PAS OUVRIR AVANT le [XX] février 2026 à 11h00"
