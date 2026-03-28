<div align="center">

# 🏭 AXEVORI
### Enterprise SAP AI Automation Platform

**Natural Language → SAP ERP · Built for India · No Competitor Has This**

[![Python](https://img.shields.io/badge/Python-3.11+-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![LangGraph](https://img.shields.io/badge/LangGraph-Multi--Agent-FF6B6B?style=flat-square)](https://langchain-ai.github.io/langgraph/)
[![Claude](https://img.shields.io/badge/Anthropic-Claude%20Sonnet-D4A017?style=flat-square)](https://anthropic.com)
[![SAP](https://img.shields.io/badge/SAP-S%2F4HANA%20%2B%20ECC%206.0-0FAAFF?style=flat-square)](https://sap.com)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docker.com)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

---

*"An employee photographs a paper invoice on WhatsApp. Minutes later it's posted to SAP FI-AP — with GST validation, manager approval, and a full audit trail. No SAP login ever touched."*

[View Architecture](#architecture) · [Features](#features) · [Quick Start](#quick-start) · [Indian Compliance](#-indian-compliance-module) · [Roadmap](#roadmap)

</div>

---

## What is Axevori?

Axevori is an **enterprise-grade, multi-agent AI platform** that connects natural language to SAP ERP systems. Employees interact via WhatsApp, Web, or Voice — Axevori handles the rest: understanding intent, validating against business rules, routing through an approval gate, and executing transactions directly in SAP.

**The problem it solves:** SAP manages 77% of the world's transaction revenue but requires months of training to use. Finance clerks, HR managers, and operations staff waste hundreds of hours per year navigating complex SAP GUIs. Axevori makes SAP accessible to everyone through natural language — while keeping full audit trails and human approval for risky operations.

**Why India first:** No global SAP AI competitor — not Microsoft Copilot, not SAP Joule, not UiPath — has built-in Indian statutory compliance. GST e-Invoice, TDS automation, PF/ESI calculations, and Professional Tax are built into Axevori's core. This is a genuine, defensible first-mover advantage.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        INPUT CHANNELS                           │
│   📱 WhatsApp    🌐 Web Chat    🎙️ Voice    📧 Email    🔗 API   │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│              🧠 MASTER ORCHESTRATOR AGENT                       │
│         LangGraph State Machine · Claude Sonnet                 │
│                                                                 │
│  Context Hydration → Intent Classification → Goal Decomposition │
│       → Parallel DAG Dispatch → Result Synthesis               │
└──────┬──────────┬──────────┬──────────┬──────────┬─────────────┘
       │          │          │          │          │
       ▼          ▼          ▼          ▼          ▼
   💰 FI/CO   👤 HR/HCM  📦 MM/SD  🏭 PP/PM  🎧 Support
   Finance    Payroll    Inventory  Production  Helpdesk
   Agent      Agent      Agent      Agent       Agent
       │          │          │          │          │
       └──────────┴──────────┴──────────┴──────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                  🚦 RISK-BASED APPROVAL GATE                    │
│  READ → Auto  |  <₹10K → Auto  |  >₹10K → Manager WhatsApp    │
│  DELETE → Admin Approval  |  Payroll → Two-Person Rule         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│             🔌 SAP CONNECTION ABSTRACTION LAYER                 │
│         Auto-detects SAP version on first connection            │
│   S/4HANA 1709+ → OData v4    |    ECC 6.0 → BAPI/RFC         │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
              ┌────────────────────────┐
              │    🏭 SAP SYSTEM       │
              │  S/4HANA · ECC · BTP  │
              └────────────────────────┘
```

### Infrastructure Stack

```
┌─────────────────────────────────────────────────┐
│                  MEMORY LAYER                   │
│  Redis (session)  ·  Qdrant (semantic memory)  │
│       PostgreSQL (history + tenant data)        │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│              3-TIER AI ROUTING                  │
│  Tier 1: Claude Haiku / Groq LLaMA  (<200ms)  │
│  Tier 2: Claude Sonnet              (<1s)      │
│  Tier 3: Claude Opus                (<5s)      │
│          60% / 35% / 5% split                  │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│               ASYNC WORKERS                     │
│      Celery + Redis · Parallel DAG execution   │
│     Docker Compose · AWS EC2/S3/RDS            │
└─────────────────────────────────────────────────┘
```

---

## Features

### 🧠 Master Orchestrator Agent
- **LangGraph state machine** — not a simple keyword router; true goal decomposition
- Decomposes complex requests into parallel sub-tasks: *"Process all overdue invoices from October"* → lists + validates + posts all invoices simultaneously
- DAG execution via Celery workers — 10 tasks in the time competitors do 3
- Full context hydration from Redis, Qdrant, and PostgreSQL before every decision
- Intent classification with Claude Haiku in <200ms

### 🚦 Risk-Based Approval Gate (5 Tiers)
| Tier | Operation | Action |
|------|-----------|--------|
| READ | Any SAP query or report | ✅ Auto-approve instantly |
| WRITE S | Write operations < ₹10K | ✅ Auto-approve with audit trail |
| WRITE L | Write operations > ₹10K | 👤 Manager WhatsApp approval (15-min timeout) |
| DELETE | Any reversal or deletion | 🔐 Admin approval required |
| PAYROLL | Payroll / bank transfers | 👥 Two-person rule — cannot be overridden |

### 🔌 SAP Connection Abstraction Layer
- **Single codebase** works on both S/4HANA and legacy ECC 6.0
- Auto-detects SAP version on first connection via `/sap/bc/rest/deploymentinfo`
- Factory pattern: `ODataConnector` for S/4HANA 1909+ · `BAPIConnector` for ECC 6.0
- All domain agents call identical API: `sap.get_invoice_list()`, `sap.post_document()`
- Standardized error objects + circuit breaker across both protocols

### 📱 WhatsApp Invoice OCR Flow
The most impressive demo for any Finance team:

```
Employee photos invoice on phone
         ↓
Claude Vision + Google Vision OCR
(vendor name, GSTIN, HSN codes, amounts, GST breakup)
         ↓
SAP vendor master validation via OData
         ↓
Indian GST compliance check
         ↓
Risk gate: ≤₹10K auto-post · >₹10K → Manager WhatsApp
         ↓
BAPI_ACC_DOCUMENT_POST → FB60 in SAP FI-AP
         ↓
"Invoice INV-2024-4521 posted. SAP Doc #1900000456, ₹48,500"
```
Zero SAP logins. Zero manual data entry. Full audit trail.

### 🏠 On-Premise SAP Connector
Solves the #1 enterprise deal-blocker — IT security teams refusing to expose SAP to the cloud:
- Customer deploys one Docker container inside their firewall
- Container connects to SAP internally via BAPI/RFC or OData
- Opens **outbound HTTPS only** (port 443) to Axevori cloud
- Zero inbound firewall rules · No VPN · Mutual TLS · Certificate pinning
- Active-passive HA — secondary activates within 30 seconds of primary failure

### 🔀 3-Tier AI Intelligence Routing
Cost-optimized routing keeps Claude Opus usage under 5% while maintaining quality:

```python
# Tier 1 — Nano (60% of tasks) — <200ms — ~$0.001/task
# Intent classification, routing, simple lookups, FAQ
# Claude Haiku / Groq LLaMA3

# Tier 2 — Mid (35% of tasks) — <1s — ~$0.003/task  
# Standard execution, SAP reads, drafting responses
# Claude Sonnet (main workhorse)

# Tier 3 — Power (5% of tasks) — <5s — ~$0.015/task
# Complex multi-step reasoning, parallel SAP ops, critical decisions
# Claude Opus
```

---

## 🇮🇳 Indian Compliance Module

**No global competitor has this.** Built in Bengaluru for the Indian market.

| Module | What it does |
|--------|-------------|
| **GST e-Invoice** | Auto-generates e-invoice JSON, submits to IRP portal, reconciles GSTR-1 & GSTR-3B, flags ITC mismatches |
| **TDS Automation** | Calculates TDS by vendor type and Section (194C, 194J, 194H), monitors thresholds, auto-generates Form 26Q |
| **Provident Fund** | Employer (12%) + employee (12%) PF contributions, ECR file for EPFO portal, UAN tracking in SAP HCM |
| **ESI (ESIC)** | Gross salary eligibility monitoring (≤₹21K/month), employer 3.25% + employee 0.75% contributions, monthly returns |
| **Professional Tax** | State-wise PT slabs — Karnataka, Maharashtra, AP/TS, Tamil Nadu — configurable per company |

---

## SAP Modules Supported

### Phase 1 (MVP) — Finance & Controlling
- **FB60** — Vendor invoice posting automation
- **F110** — Payment run scheduling and automation
- **GL** — General ledger reconciliation in natural language
- **CO** — Cost center reporting: *"Show Q1 Marketing spend vs budget"*
- **GST/TDS** — Full Indian tax compliance on every transaction

### Phase 2
- **HR/HCM** — Leave management, payslips, headcount reports
- **MM** — Purchase orders, vendor queries, GRN status
- **SD** — Sales orders, delivery status, customer AR

### Phase 3
- **PP/PM** — Production planning, plant maintenance
- **SAP HANA** — Direct HANA view queries for complex analytics
- **SAP BTP** — Event-driven automation across multi-system flows

---

## Tech Stack

| Layer | Technologies |
|-------|-------------|
| **AI/LLM** | LangGraph · Claude Haiku/Sonnet/Opus · Groq LLaMA3 |
| **RAG/Memory** | Qdrant (semantic) · Redis (session) · PostgreSQL (history) |
| **OCR** | Claude Vision · Google Vision API |
| **Backend** | Python 3.11+ · FastAPI · Django REST Framework · Celery |
| **SAP** | OData v4 · BAPI/RFC · SAP JCo Connector |
| **Frontend** | React TypeScript · Chart.js |
| **Database** | PostgreSQL (pgvector + RLS) · Redis · MySQL |
| **Security** | JWT · RBAC · Mutual TLS · Certificate pinning · OWASP |
| **DevOps** | Docker Compose · AWS (EC2/S3/RDS) · CI/CD · GitHub Actions |
| **Channels** | WhatsApp Business API · Meta Cloud API · Twilio |

---

## Quick Start

### Prerequisites
- Python 3.11+
- Docker & Docker Compose
- SAP S/4HANA or ECC 6.0 instance (or sandbox)
- Anthropic API key
- WhatsApp Business API credentials (Meta Cloud API or Twilio)

### 1. Clone and configure

```bash
git clone https://github.com/karthikkarthi-Ghost/axevori.git
cd axevori
cp .env.example .env
```

### 2. Set environment variables

```env
# AI Models
ANTHROPIC_API_KEY=your_key_here
GROQ_API_KEY=your_key_here

# SAP Connection
SAP_HOST=your_sap_host
SAP_CLIENT=100
SAP_USERNAME=axevori_service_user
SAP_PASSWORD=your_password
SAP_VERSION=auto  # auto-detects S/4HANA or ECC

# WhatsApp
WHATSAPP_TOKEN=your_meta_token
WHATSAPP_PHONE_ID=your_phone_id
WEBHOOK_VERIFY_TOKEN=your_verify_token

# Databases
POSTGRES_URL=postgresql://user:pass@localhost:5432/axevori
REDIS_URL=redis://localhost:6379
QDRANT_URL=http://localhost:6333

# Indian Compliance
GST_API_KEY=your_gstn_api_key
IRP_ENDPOINT=https://einvoice1.gst.gov.in
```

### 3. Start with Docker Compose

```bash
docker compose up -d
```

This starts: FastAPI backend · Celery workers · Redis · PostgreSQL · Qdrant

### 4. Test the WhatsApp webhook

```bash
# Verify webhook
curl -X GET "https://your-domain.com/webhook?hub.verify_token=YOUR_TOKEN&hub.challenge=test"

# Send a test message (simulate WhatsApp)
curl -X POST http://localhost:8000/api/v1/message \
  -H "Content-Type: application/json" \
  -d '{"from": "919663007653", "text": "Show me all open invoices above 50000"}'
```

### On-Premise Connector (for enterprise customers)

```bash
# Customer runs this inside their firewall — one command, 5 minutes
docker pull axevori/sap-connector
docker run -d \
  --name axevori-connector \
  -e TENANT_TOKEN=your_tenant_token \
  -e SAP_HOST=internal-sap-host \
  -e SAP_CLIENT=100 \
  axevori/sap-connector
```

No inbound firewall rules needed. Outbound HTTPS port 443 only.

---

## Project Structure

```
axevori/
├── core/
│   ├── orchestrator/          # LangGraph state machine
│   │   ├── graph.py           # Main LangGraph workflow
│   │   ├── intent.py          # Claude Haiku intent classifier
│   │   └── decomposer.py      # Goal → parallel tasks
│   ├── agents/
│   │   ├── fi_co_agent.py     # Finance & Controlling
│   │   ├── hr_agent.py        # HR/HCM
│   │   ├── mm_agent.py        # Materials Management
│   │   └── support_agent.py   # General support
│   ├── approval_gate/
│   │   ├── risk_classifier.py # 5-tier risk assessment
│   │   └── whatsapp_approval.py # Manager approval flow
│   └── routing/
│       └── model_router.py    # 3-tier AI routing logic
├── sap/
│   ├── abstraction.py         # SAP version auto-detection
│   ├── odata_connector.py     # S/4HANA OData v4
│   ├── bapi_connector.py      # ECC 6.0 BAPI/RFC
│   └── on_premise/
│       └── connector/         # Docker on-premise agent
├── compliance/
│   ├── gst.py                 # GST e-Invoice & filing
│   ├── tds.py                 # TDS automation
│   ├── pf.py                  # Provident Fund (EPFO)
│   ├── esi.py                 # ESI (ESIC)
│   └── professional_tax.py   # State-wise PT
├── channels/
│   ├── whatsapp.py            # WhatsApp Business API
│   ├── web.py                 # Web chat interface
│   └── ocr_pipeline.py        # Invoice photo → SAP
├── memory/
│   ├── redis_session.py       # Session context
│   ├── qdrant_store.py        # Semantic memory
│   └── postgres_history.py    # Conversation history
├── api/
│   └── v1/                    # FastAPI REST endpoints
├── workers/
│   └── celery_tasks.py        # Parallel async workers
├── frontend/                  # React TypeScript dashboard
├── docker-compose.yml
└── .env.example
```

---

## Security

- **Multi-tenant isolation** — PostgreSQL Row-Level Security (RLS), every query is tenant-scoped
- **JWT + RBAC** — role-based access enforced at API and SAP operation level
- **Mutual TLS** — on-premise connector uses certificate pinning, prevents MITM
- **Approval gate** — no AI agent can write to SAP without human approval above threshold
- **OWASP compliant** — input sanitization, parameterized queries, secure headers
- **Secret management** — zero hardcoded credentials, all via environment variables
- **Audit trail** — every SAP operation logged permanently with reasoning chain

---

## Roadmap

- [x] LangGraph orchestrator with parallel DAG execution
- [x] Risk-based 5-tier approval gate
- [x] SAP abstraction layer (S/4HANA + ECC 6.0)
- [x] WhatsApp channel + Invoice OCR flow
- [x] Indian compliance module (GST, TDS, PF, ESI, PT)
- [x] On-premise SAP connector (Docker, outbound HTTPS)
- [x] 3-tier AI model routing
- [ ] Voice channel (Bhashini API for regional languages)
- [ ] SAP MM/SD agent (Phase 2)
- [ ] LangSmith observability integration
- [ ] Fine-tuned intent classifier on Indian SAP terminology
- [ ] Mobile app (React Native)
- [ ] SAP BTP event-driven automation (Phase 3)

---

## Why Axevori Wins in India

| Competitor | Indian GST | TDS | PF/ESI | On-Premise | WhatsApp |
|-----------|-----------|-----|--------|------------|----------|
| Microsoft Copilot | ❌ | ❌ | ❌ | ❌ | ❌ |
| SAP Joule | ❌ | ❌ | ❌ | ❌ | ❌ |
| UiPath | ❌ | ❌ | ❌ | ✅ | ❌ |
| **Axevori** | **✅** | **✅** | **✅** | **✅** | **✅** |

---

## Author

**Karthik A N** — AI Engineer · Full Stack Developer · Built in Bengaluru 🇮🇳

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/in/a-n-karthik-92832722a)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat-square&logo=github)](https://github.com/karthikkarthi-Ghost)
[![Email](https://img.shields.io/badge/Email-Contact-EA4335?style=flat-square&logo=gmail)](mailto:karthik3004an@gmail.com)

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

<div align="center">

**⭐ Star this repo if you found it useful**

*Built with LangGraph · Claude · SAP · ❤️ from Bengaluru*

</div>
