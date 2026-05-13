# 🏛️ Enterprise Systems Architecture Dossier
**Document ID**: `ARCH-MASTER-2026`  
**Lead Systems Architect & Author**: Steinley Goh  
**Scope**: Full-Stack Operational Infrastructure, Multi-Dealer Messaging Flow, Database Integrity, Hardware Telemetry

---

## 📑 Document Control & Overview
This technical manual outlines the rigorous logic, state machines, and API integration flows designed to transition regional marketing and customer relationship operations from highly manual legacy dependencies into autonomous, scalable infrastructures.

---

## ⚡ Module 1: Multi-Dealer WhatsApp Distribution Engine (`TRD-101`)

### **1.1 Gateway Topography**
Interconnected the **Official Yadea WhatsApp Business API** (Meta Cloud) with the **Mekari Qontak** messaging core. Webhooks parse asynchronous user communication JSON payloads to extract conversation triggers automatically.

### **1.2 Geographic Intent & Chatbot Screening**
Incoming traffic flows through an automated conversational pre-screening node to capture location coordinates and operational variables.
* **Routing Algorithm**: Queries customer latitude/longitude inputs against verified regional dealership databases.
* **Distribution Cap**: Automatically routes qualified payloads to the **three nearest authorized retail locations** simultaneously.

### **1.3 Competition Mode State Machine ("First-to-Respond")**
To enforce rapid response SLAs and prevent sales personnel from cherry-picking leads, a thread-safe **Competition Mode** distribution protocol executes via atomic DB checks:
```
[ Incoming Lead Message Payload ]
               │
               ▼
   [ DB State Set: UNCLAIMED ] ───► [ Broadcast Interactive Push to Reps A, B, C ]
               │
               ▼
[ Intercept Rep Button Tap Callback ]
               │
               ├─► If State == UNCLAIMED:
               │        │
               │        ├─► Execute DB Update: State = CLAIMED (Lock Acquired)
               │        ├─► Route Full Customer Context to Winning Rep
               │        └─► Append Handoff Timestamp to CRM Ledger
               │
               └─► If State == CLAIMED:
                        │
                        └─► Fire Async Webhook: Send "❌ Too Slow" Interface Notice
```

---

## 🛡️ Module 2: CRM Integrity, Fraud Prevention & Alert Tiering (`BRD-201`)

### **2.1 Asynchronous Data Validation**
Operating concurrently across **110+ individual dealer portals**, internal triggers actively enforce compliance to protect unit margin economics:
* **Serial Number Validation**: Automated ingestion hooks cross-reference submitted unit serial numbers against corporate registration ledgers to prevent warranty fraud and duplicate commission claims.
* **MAP Threshold Checks**: Background pricing audit scripts evaluate regional sell-through figures to immediately detect unauthorized regional discounting.

### **2.2 Alert Tiering Framework**
To prevent notification blindness among executive staff, message bus traffic is categorized into three strict tiers:
1. **P0 (Critical)**: Dispatched natively via WhatsApp push alerts directly to regional management for immediate SLA bottlenecks.
2. **P1 (Warning)**: Surfaced dynamically within the custom administrative web interface to track recurring serial anomalies.
3. **P2 (Info)**: Aggregated asynchronously into daily formatted Markdown/HTML email recaps outlining regional conversion health.

---

## 📊 Module 3: Unit Economics Analytics Engine (`ROI-301`)

### **3.1 Visual Analytics Core**
Configured customized visual interface matrices inside **Feishu Base** and **Google Looker Studio** built upon an absolute mathematical proof linking operational burn directly to unit-level customer acquisition momentum.

### **3.2 The 10-Variable Dynamic Ledger**
The analytical model ingests, computes, and charts 10 core operational vectors alongside multi-channel performance comparisons (benchmarking TikTok performance, Google Search volume, and unassigned physical walk-ins):
1. **Total Event Spend**: Overall capital outlay deployed per localized marketing initiative.
2. **Dealer Support Cost**: Internal subsidies allocated from headquarters to sustain regional operations.
3. **Baseline Dealer Cost**: Fixed infrastructural maintenance mapping to specific partner nodes.
4. **Out-of-Pocket Dealer Spend**: Unsubsidized capital deployed independently by regional dealers.
5. **Average Cost per Activity**: Efficiency metric comparing overall spend against isolated event footprints.
6. **Average Cost per Day**: Dynamic burn rate calculated across ongoing active operational pushes.
7. **Cost-Per-Lead (CPL) Trends**: Granular tracking mapping absolute monthly CPL metrics alongside interactive daily trendline distributions.
8. **Total Activity Volume**: Cumulative count of strategic events launched.
9. **Active Duration Days**: Combined temporal operational footprint measured in campaign lifecycles.
10. **Verified Sell-Through Outcomes**: Absolute monthly lead generation outputs cross-audited against finalized vehicle purchase registrations.

---

## 🔗 Module 4: Hardware Telemetry Integration Architecture (`IoT-401`)

### **4.1 Native Figma Layout Blueprint**
Designed the comprehensive **12-24 Month System Architecture Roadmap** natively inside **Figma** (including highly specialized vendor alignment interface views crafted `For mekari To view`) to standardize communication flows between software CRM entities and smart hardware units.

```
[ Finalized Hardware Point-of-Sale ]
                │
                ▼
  [ Trigger Embedded IoT Telemetry ]
                │
                ▼
   [ Dispatch Async API Handshake ]
                │
                ▼
  [ Update Central Hub Ledger State ]
```

### **4.2 Execution Protocols**
* Programmed database handshakes to interface securely with smart embedded hardware reporting vehicle status updates.
* Configured automated event logic triggering centralized ledger updates the precise millisecond a localized unit registers as officially purchased.
* Validated that complete layout definitions mapped natively inside Figma combined with crisp logic loops secure rapid executive consensus without heavy upfront enterprise custom coding.

---

## 📁 Module 5: Standalone System Components
For immediate production deployment, refer directly to companion operational repositories:
* **[`LEAD_INTAKE_ENGINE.md`](./LEAD_INTAKE_ENGINE.md)**: Drop-in Google Apps Script automation core handling deduplication and webhook dispatch.
* **[`PRODUCTIZED_OPS_KIT.md`](./PRODUCTIZED_OPS_KIT.md)**: Instant operational frameworks mapping spreadsheet column configurations.
* **[`WHATSAPP_COMPETITION_ROUTER.md`](./WHATSAPP_COMPETITION_ROUTER.md)**: Standalone multi-agent distribution logic specifications.
