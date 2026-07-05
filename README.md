# Cloud Infrastructure & Distributed Home Lab Cluster

An automated, lightweight, multi-service home lab infrastructure blueprint engineered for resource-constrained cloud compute nodes. This cluster uses an orchestration mesh to safely publish essential private utilities through a Zero-Trust edge architecture.

## 🏗️ Architectural Overview

The deployment pattern transitions away from heavy logging/computation frameworks to optimize a constrained memory boundary (**2 vCPU / 12 GB RAM**). Services are grouped into dynamic operational layers isolated inside Docker network definitions.

```text
       [ Public Internet ]
                │ (Future State via Cloudflare Edge)
                ▼
       [ Caddy Reverse Proxy ]
                │
         ┌──────┴──────────────────────────┐
         ▼ (Tailscale Mesh / Internal Net) ▼
   ┌───────────┐                     ┌───────────┐
   │  Casdoor  │                     │ OwnTracks │
   │  (Auth)   │                     │(Telemetry)│
   └───────────┘                     └───────────┘
         │                                 │
         ▼                                 ▼
   ┌───────────┐                     ┌───────────┐
   │  RustDesk │                     │  Uptime   │
   │  (Relay)  │                     │   Kuma    │
   └───────────┘                     └───────────┘
```

## 🔐 Core Service Mesh

1. **Identity & Access Management Gate:** Driven by `Casdoor` to handle programmatic Zero-Trust user federation at the cluster boundary.
2. **Secure Remote Access Relay:** Powered by `RustDesk Server` (`hbbs` / `hbbr`) acting as a self-hosted signaling plane to coordinate peer-to-peer tunnels without public NAT traversal issues.
3. **Telemetry & Geofenced Logging:** Managed by `OwnTracks Recorder` deployed over an on-demand stateless HTTP pipeline to track mobile telemetry metrics efficiently without socket-induced battery drain.
4. **Resiliency Engine:** Monitored continuously by lightweight heartbeats via `Uptime Kuma`.

---

## 🛠️ Infrastructure Configuration

### 1. Network & Topology Layout
During the bootstrap phase, all containers sit isolated behind a private `Tailscale` WireGuard overlay mesh network. 

Authoritative `NS` delegation to an independent Cloudflare tenant is staged to handle:
* Wildcard subdomain mapping (`*.subdomain.domain`) routing asset paths directly to root contexts (`/`).
* Automated `DNS-01` ACME challenge verification for dynamically provisioned SSL/TLS certificates.

### 2. Micro-Resource Management & Anti-Reclamation
To prevent automated cloud provider node reclamation flags while safeguarding system availability, resource constraints are dynamically enforced:
* **Memory Protection:** Managed via a target balancing daemon script tracking active memory allocation pools inside `/proc/meminfo`. It maintains a safe ~25% baseline system usage boundary without context starvation.
* **CPU Protection:** Regulated by percentage-driven system control limits (`lookbusy`) to scale consumption automatically to host capacity variations.

---

## 🚀 Deployment Strategy

The environment is provisioned via an automated master restoration controller (`SETUP_MASTER.sh`) ensuring consistent state storage allocation and strict environment variables reproducibility.

### Prerequisites
* Docker Engine v24.x+ & Compose V2 plugin
* Configured Tailscale interface daemon

### Volume Structure
```bash
/docker/
  ├── casdoor_data/
  ├── owntracks_store/
  ├── owntracks_config/
  └── rustdesk_data/
```

### Initial Node Bootstrap
```bash
# Clone infrastructure blueprint
git clone https://github.com/your-repo/infrastructure-cluster.git
cd infrastructure-cluster

# Set script execute bits and trigger deployment
chmod +x scripts/SETUP_MASTER.sh
./scripts/SETUP_MASTER.sh
```

---

## 📋 Post-Deployment Validation Tasks

Following automated script executions, the following tasks are manually initiated to lock configuration states:
1. Bind unique Tailscale Node IP allocations inside the `docker-compose.yml` environment blocks.
2. Extract the auto-generated RustDesk asymmetric public key payload:
   ```bash
   cat docker/rustdesk_data/id_ed25519.pub
   ```
3. Establish programmatic webhook receivers to route critical node failure logs down to endpoints.
