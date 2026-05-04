# 📋 Functional Requirements Document — Fleet Dashboard & OTA/AWS
## (For Customer Confirmation)

> **References:**
> - Fleet_dashboard_v0 (1).pdf — Fleet Dashboard Feature Proposal
> - 선박용_OTA_AWS_요구기능_정리.pdf — Ship OTA/AWS Functional Requirements
>
> **System:** SMTAI HMS Fleet Dashboard
> **Date:** 2026-04-23
> **Author:** BA Team

---

## 📌 Document Purpose

This document consolidates **all functional requirements** from the two customer documents, categorized by module. Each requirement is accompanied by **confirmation questions** to be clarified before proceeding with design and development.

---

## I. SYSTEM OVERVIEW

**Description:** A web platform for remote maritime safety monitoring, fleet management, event processing, and OTA software deployment for onboard edge devices.

**Technical Premises (from original documents):**
- No real-time video streaming → The system operates on an **event-driven** model (viewing clips after an event occurs).
- Cloud Infrastructure: AWS (IoT Greengrass, MQTT, IoT Jobs).
- Edge devices on ships run container-based services.

**2 Main User Groups:**

| Group | Target Audience | Permissions |
|------|-----------|-----------|
| **Admin** | Developers (Dev) + Operations Staff (Ops) | Full Access — Dev has the highest privileges |
| **End-User** | Ship Owners / Safety Officers | Limited Access — Monitoring & viewing reports only |

### ❓ Confirmation Questions — Overview

| # | Question | Reason |
|---|---------|----------------|
| Q1 | Dev and Ops **share the same dashboard** but have different permissions — is this correct? Specifically, what are the limitations for Ops compared to Dev? | To design the RBAC matrix |
| Q2 | Do End-users (ship owners) need **separate accounts** to log in, or will they only be provided with a link to view reports? | To determine if user management is needed for end-users |
| Q3 | Does the system need to support **multi-tenancy**? (1 instance serving multiple ship owner companies) | Affects data isolation architecture |

---

## II. DETAILED FUNCTIONAL REQUIREMENTS

### Module 1: Fleet Overview

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-1.1 | AIS Map | Both files | Display real-time ship locations on a map based on AIS data |
| FR-1.2 | Route Display | OTA/AWS | Display ship routes on the map |
| FR-1.3 | Ship Connection Status | Both files | Online/Offline, last sync time, latency |
| FR-1.4 | Fleet Selection | Fleet Dashboard | Screen to select/switch between different fleets |
| FR-1.5 | Ship Summary | Fleet Dashboard | Number of cameras, number of events, alarm status per ship |
| FR-1.6 | Hierarchical Management | OTA/AWS | Customer → Fleet → Ship (multi-level hierarchy) |

### ❓ Confirmation Questions — Fleet Overview

| # | Question | Reason |
|---|---------|-------|
| Q4 | Where does the AIS data come from? A 3rd-party API or self-collected from onboard devices? | Determines the architecture of the map component |
| Q5 | Has the feasibility of the AIS Map and Route Display features been **confirmed**? (The OTA/AWS document notes "needs confirmation") | Decides whether to include it in the MVP |
| Q6 | What is the expected number of ships managed simultaneously? (5? 50? 500?) | Affects performance & UI pagination |
| Q7 | How many fleets can 1 customer own? | Determines navigation hierarchy |

---

### Module 2: Event Management

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-2.1 | Event List | Both files | Event list with filter/search, categorized by severity |
| FR-2.2 | Event Details | Both files | View detailed info: time, type, camera, ship |
| FR-2.3 | Event Video Clip | Both files | Playback video before/after the event time |
| FR-2.4 | Event Images | Both files | View related captured images |
| FR-2.5 | Event Status | Fleet Dashboard | Lifecycle: New → In Progress → Resolved |
| FR-2.6 | Post-event Processing | Fleet Dashboard | Record handling measures, save evidence |
| FR-2.7 | Event Trend | Fleet Dashboard | Event trend chart over time (24h/7d) |
| FR-2.8 | Data Transmission by Severity | OTA/AWS | Fire/smoke → send video+images **immediately**; Other events → send images+metadata immediately |
| FR-2.9 | Search & Filter Events | Fleet Dashboard | Filter by ship, time, event type; Bookmark/Favorites |

### ❓ Confirmation Questions — Event Management

| # | Question | Reason |
|---|---------|-------|
| Q8 | What are the statuses in the event lifecycle? (New → Acknowledged → In Progress → Resolved → Closed?) Who changes the status? | To design the event state machine |
| Q9 | "Post-event processing" — does it require an approval workflow? Or just text logging? | Determines workflow complexity |
| Q10 | For **fire/smoke** events that send video immediately — how many seconds is the video? How long before and after the event? | Specs for video clip extraction |
| Q11 | HiCAMS storing video/images for data collection is noted as "develop later" — do we need to place a placeholder in the UI? | Decides whether to design for it now |

---

### Module 3: Monitoring & Edge Health

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-3.1 | System Status Dashboard | Both files | Server, network, and service status for each ship |
| FR-3.2 | Edge Device Monitoring | Fleet Dashboard | CPU/GPU/Memory/Storage on the edge device |
| FR-3.3 | Camera Connection Status | Both files | Online/Offline status for each camera |
| FR-3.4 | Alarm Board | Both files | Alarms for detected anomalies (deployment/operations) |
| FR-3.5 | Bandwidth & Latency | Both files | Monitor network bandwidth per ship |
| FR-3.6 | Error Message Viewer | OTA/AWS | Display error messages from ships — for non-dev personnel |

### ❓ Confirmation Questions — Monitoring

| # | Question | Reason |
|---|---------|-------|
| Q12 | How often is Edge Health data (CPU/GPU/Memory) polled? Real-time or every 5 minutes? | Specs for monitoring interval |
| Q13 | Warning thresholds for CPU/Memory/Storage — does the customer have specific values? Or should users configure them? | To design the alert rule engine |
| Q14 | "Non-dev personnel can easily deploy/debug" — who exactly are the non-dev personnel? Operators at the onshore center? Or the ship owner's IT staff? | Determines the level of UI simplification |

---

### Module 4: OTA Deployment

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-4.1 | Deploy Controller | Both files | Deploy at 3 levels: bulk / fleet / individual ship |
| FR-4.2 | Deployment Status | Both files | Progress, success, failed per ship |
| FR-4.3 | Deployment History | Both files | History with filters by ship/date/result |
| FR-4.4 | Rollback | Both files | Revert to previous version if deployment fails |
| FR-4.5 | Staged Rollout | Fleet Dashboard | Gradual deployment: pilot → batch → full fleet |
| FR-4.6 | Version Matrix | Fleet Dashboard | Display version matrix per ship |
| FR-4.7 | Adaptive Bandwidth Download | OTA/AWS | Detect bandwidth → decide to download/stop |
| FR-4.8 | Separate Download / Deploy | OTA/AWS | Finish image download → user confirms → then deploy |
| FR-4.9 | New Ship Registration | OTA/AWS | Onboarding: register & setup new ships in the system |
| FR-4.10 | Configuration Deployment | Fleet Dashboard | Deploy config changes (not just software) |

### ❓ Confirmation Questions — OTA

| # | Question | Reason |
|---|---------|-------|
| Q15 | How many confirmation steps are needed for deployment? (Download → Confirm → Deploy → Verify?) | To design the wizard flow |
| Q16 | Automatic rollback on health check failure? Or manual rollback? | Specs for rollback strategy |
| Q17 | "Staged rollout" — what are the proportions? (10% → 50% → 100%?) Who decides to transition between stages? | Specs for staged rollout |
| Q18 | Secure tunneling for emergencies — what is the specific scope? SSH into the edge device? Or just send commands? | Determines security boundaries |

---

### Module 5: Remote Management

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-5.1 | Remote Camera Settings | Fleet Dashboard | Configure camera parameters |
| FR-5.2 | Remote AI Model Config | OTA/AWS | Change model params (e.g., decision_threshold) |
| FR-5.3 | Remote DB Variable Config | OTA/AWS | Change DB variables from the cloud |
| FR-5.4 | Remote Diagnostics | Fleet Dashboard | Detailed logs, Log Explorer, remote commands |
| FR-5.5 | Portable Camera Connection | Fleet Dashboard | Link portable cameras to the system |

### ❓ Confirmation Questions — Remote Management

| # | Question | Reason |
|---|---------|-------|
| Q19 | Do remote config changes (model, DB) require an **audit log**? Is approval needed? | To design remote config + audit trail |
| Q20 | Remote Terminal (document notes "not usual") — should this be included in the scope or removed? | Determines scope |
| Q21 | Portable cameras — what type of camera? Connection protocol? (RTSP? ONVIF? USB?) | Specs for camera integration |

---

### Module 6: Statistics & Reports

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-6.1 | Safety Score | Fleet Dashboard | Score the safety level of each ship |
| FR-6.2 | Event Statistics | Fleet Dashboard | Event stats by day/week (24h/7d trend) |
| FR-6.3 | Event Resolution Rate | Fleet Dashboard | % of events processed vs unprocessed |
| FR-6.4 | Monthly Report | Fleet Dashboard | Periodic comprehensive report |
| FR-6.5 | Fleet Statistics | Fleet Dashboard | Compare performance between ships |

### ❓ Confirmation Questions — Statistics

| # | Question | Reason |
|---|---------|-------|
| Q22 | What is the formula for calculating the **Safety Score**? What metrics is it based on? Red/Yellow/Green thresholds? | Exact specs for scoring algorithm |
| Q23 | Monthly Report — what format? (PDF export? Dashboard view? Automated email?) | Determines report output format |
| Q24 | Is there a need to **compare** Safety Scores between ships / between months? | Determines required chart types |

---

### Module 7: MLOps (Admin only)

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-7.1 | False-positive Data Collection | Fleet Dashboard | Collect overkill/miss data to retrain the model |
| FR-7.2 | Model Update Status | Fleet Dashboard | Track model training/deployment status |
| FR-7.3 | Model Version History | Fleet Dashboard | Model version history & changelog |
| FR-7.4 | Periodic Data Collection | Fleet Dashboard | Scheduled data collection pipeline |

### ❓ Confirmation Questions — MLOps

| # | Question | Reason |
|---|---------|-------|
| Q25 | Is MLOps within the MVP scope or a later Phase? | Priority alignment |
| Q26 | Data collection pipeline — is data sent to AWS S3? Or separate storage? | Specs for data architecture |

---

### Module 8: Customer & Admin Management

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-8.1 | Customer CRUD | OTA/AWS | Register, edit, delete customers |
| FR-8.2 | Fleet / Ship CRUD | OTA/AWS | Manage fleets & ships per customer |
| FR-8.3 | Permissions | Both files | RBAC: Admin/Operator/Viewer |
| FR-8.4 | Installation Progress | Fleet Dashboard | Tracking installation progress for new ships |
| FR-8.5 | Audit Log | Fleet Dashboard | History of setting changes, video views |

### ❓ Confirmation Questions — Admin

| # | Question | Reason |
|---|---------|-------|
| Q27 | How long are audit logs retained? Are there compliance requirements? (IMO regulations?) | Data retention policy |
| Q28 | How many permission roles are there? Only 3 (Admin/Operator/Viewer) or are custom roles needed? | RBAC complexity |

---

### Module 9: Support & Communication

| REQ-ID | Requirement | Source | Description |
|--------|---------|-------|-------|
| FR-9.1 | Ship ↔ Shore Chat | Fleet Dashboard | Communication between crew and onshore supervisors |
| FR-9.2 | Mobile Notification | Fleet Dashboard | Push notification for end-users when events occur |

### ❓ Confirmation Questions — Support

| # | Question | Reason |
|---|---------|-------|
| Q29 | Chat — text only or file/image sending needed? Do chat histories need to be saved? | Scope for chat feature |
| Q30 | Mobile notification — separate app or integrated into an existing app? What events trigger it? | Platform decision |

---

## III. CONFLICTS / ITEMS TO CLARIFY

> [!WARNING]
> The following points show **inconsistencies** between the two documents or are marked as questionable:

| # | Issue | Details | Question |
|---|--------|----------|---------|
| ⚠️1 | **Real-time video feed** | OTA/AWS document marks this **(x)** — might be excluded. But Fleet Dashboard still has a Video & Data module | **Q31:** Is real-time video streaming **officially removed** or just **postponed**? |
| ⚠️2 | **Video transmission to shore** | OTA/AWS requires "transmit video to onshore center" but Fleet Dashboard states "no realtime streaming" | **Q32:** Need to distinguish clearly: immediate video transmission for fire/smoke vs. continuous video streaming — what is the exact scope? |
| ⚠️3 | **AIS Map Feasibility?** | OTA/AWS notes "needs feasibility confirmation" for AIS Map & Route Display | **Q33:** Has the feasibility check for AIS integration yielded results yet? |

---

## IV. PROPOSED PHASE BREAKDOWN

| Phase | Proposed Scope | Notes |
|-------|---------------|---------|
| **MVP** | Monitoring Dashboard, OTA Deployment, Event Management (Basic), Edge Health, RBAC | Core value — operational capability |
| **Phase 2** | Fleet Overview (AIS Map), Remote Config, Customer/Fleet/Ship CRUD, Statistics | Expanded management |
| **Phase 3** | Safety Score, Monthly Report, MLOps, Chat, Mobile Notification, Portable Camera | Enhanced experience |

### ❓ Confirmation Questions — Phasing

| # | Question | Reason |
|---|---------|-------|
| Q34 | Does the customer agree with the proposed phase breakdown? Are there features that need to be **moved to MVP** or **pushed to later phases**? | Align priorities |
| Q35 | What is the desired timeline for the MVP? | Affects resource planning |

---

## V. COMPILED QUESTIONS (Checklist for Confirmation)

### 🔴 CRITICAL Questions (Must answer before starting)

| # | Question | Module |
|---|---------|--------|
| Q1 | Dev vs Ops — what specific permission differences? | Overview |
| Q5 | AIS Map & Route Display feasibility confirmed? | Fleet Overview |
| Q6 | Number of simultaneously managed ships? | Fleet Overview |
| Q8 | Event lifecycle statuses? Who transitions them? | Event |
| Q14 | Who exactly are "non-dev personnel"? | Monitoring |
| Q22 | Safety Score formula? | Statistics |
| Q31 | Realtime video — removed or postponed? | Conflicts |
| Q34 | Agree with phase breakdown? | Phasing |
| Q35 | MVP Timeline? | Phasing |

### 🟡 IMPORTANT Questions (Must answer before UI design)

| # | Question | Module |
|---|---------|--------|
| Q3 | Multi-tenant? | Overview |
| Q4 | AIS data source? | Fleet Overview |
| Q10 | Event video clip — how many seconds? | Event |
| Q15 | OTA deploy — how many steps? | OTA |
| Q16 | Automatic or manual rollback? | OTA |
| Q19 | Remote config needs audit log? | Remote |
| Q28 | How many permission roles? | Admin |
| Q32 | Scope of video transmission to shore? | Conflicts |

### 🟢 NICE-TO-HAVE Questions (Can answer later)

| # | Question | Module |
|---|---------|--------|
| Q11 | HiCAMS placeholder? | Event |
| Q20 | Remote Terminal — in scope? | Remote |
| Q21 | Portable camera protocol? | Remote |
| Q23 | Report format? | Statistics |
| Q25 | MLOps in MVP? | MLOps |
| Q29 | Chat scope? | Support |
| Q30 | Separate mobile app? | Support |

---

> [!TIP]
> **Usage Instructions:** Print this document → Take it to the customer meeting → Go through each module → Check off answered questions → Record the answers → Update the document after the meeting.
