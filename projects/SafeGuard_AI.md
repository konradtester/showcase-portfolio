# SafeGuard AI | Enterprise-Grade Facility Safety & Emergency Management

## 🚀 Business Overview

SafeGuard AI is a mission-critical safety management system designed for industrial facilities, shopping malls, and corporate offices. It provides a real-time bridge between facility managers and employees during emergency situations (Fire, SOS, Hazards). The platform ensures 100% auditable safety protocols and real-time personnel tracking during evacuations.

### Key Value Propositions:

- **Real-Time Muster Sessions**: Live tracking of evacuation progress via mobile apps.
- **GDPR (RODO) Compliance**: Privacy-first design with auditable data access.
- **Multi-Level RBAC**: Complex permission matrix for Global Owners, Facility Admins, and Coordinators.

---

## 🏗️ Engineering Challenges & Logical Solutions

### 1. The "Join Flow" Security Paradox

**Challenge**: High-security facilities need to control access strictly, but employees need a frictionless way to join the system during onboarding.
**Solution**: Created a hybrid **QR + Join Code** system. Tokens are cryptographically generated and can be rotated by admins. Users entering via code are placed in a "Pending" state, requiring manual approval in the Dashboard before sensitive data is exposed.

### 2. Auditable Support Access

**Challenge**: How can Global Admins provide technical support without having permanent access to private facility data?
**Solution**: Built a **Request-Response Support Flow**. A Global Admin must request temporary access for a specific reason. A Facility Owner must manually approve it. The access is time-bound and every action taken is logged in an immutable audit trail.

---

## 🛠️ Technology Stack

- **Backend**: FastAPI (Python) with SQLAlchemy 2.0 (PostgreSQL).
- **Frontend**: Flutter (Mobile) & React (Web Panel).
- **Communication**: Real-time WebSockets for instant alarm propagation.
