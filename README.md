<div align="center">

# LabWeb

### Laboratory Operations, Attendance & Asset Management

**Karunya Institute of Technology and Sciences**

A role-based web platform for managing laboratory attendance, occupancy, equipment requests, and operational records through one connected workflow.

<br>

[Overview](#overview) · [Features](#key-capabilities) · [Architecture](#technology-stack) · [Setup](#getting-started) · [Database](#google-sheets-database) · [Security](#security-and-data-integrity)

</div>

---

## Overview

**LabWeb** is a web-based laboratory monitoring and management system designed for Karunya Institute of Technology and Sciences. It brings attendance tracking, laboratory occupancy, course-level attendance review, equipment inventory, booking requests, and equipment issue/return workflows into a single application.

The MVP is designed around five distinct user roles, with access determined by each user's responsibilities and resource assignments. Google Apps Script handles backend operations and validation, while Google Sheets serves as the initial data store.

The project prioritizes dependable core workflows over feature volume. A feature is considered complete only when the user action, backend validation, resulting state, and persisted record work together—not merely when a dashboard or screen loads.

> **MVP principle:** Build and verify the mandatory P0 workflows first. P1 features follow only after the P0 acceptance tests pass.

## At a Glance

| Area | Project baseline |
|---|---|
| Institution | Karunya Institute of Technology and Sciences |
| Application | Web-based laboratory management system |
| Frontend | HTML, CSS/Tailwind CSS, JavaScript |
| Backend | Google Apps Script |
| Initial data store | Google Sheets |
| Authentication | Institutional Google account; approved `@karunya.edu` and `@karunya.edu.in` domains |
| Access model | Role-based access control (RBAC), enforced by the backend |
| Supported roles | Student, Faculty, Coordinator, HOD, Super Admin |
| Initial release | Two-day MVP baseline; demonstration workload |

## The Problem LabWeb Addresses

Laboratory operations involve more than recording who entered a room. Attendance, occupancy, course rosters, equipment availability, bookings, and physical handovers need to agree with one another.

LabWeb is designed to connect those activities while preserving clear permissions and a traceable history:

- Record laboratory entry and exit through QR-based attendance.
- Apply geofence validation where it is implemented and tested.
- Derive live occupancy from valid active attendance sessions.
- Compare course enrolments with qualifying attendance records.
- Track equipment inventory, requests, approvals, physical loans, and returns.
- Present personal, course, laboratory, and departmental information according to role.
- Reject unauthorized, duplicate, invalid, and conflicting operations on the server.

## Key Capabilities

### 1. Institutional Authentication & Role-Based Access

- Restrict access to approved institutional email domains.
- Provide separate dashboards and permissions for all five roles.
- Validate identity, account status, role, and resource scope in backend operations.
- Prevent users from assigning themselves elevated privileges.
- Keep protected data and operations inaccessible even when a request is made outside the normal interface.

### 2. QR Attendance & Location Validation

- Let students select a laboratory and visit reason, then scan a QR code to check in.
- Create an active attendance session after the required checks pass.
- Close the session on a valid check-out.
- Reject invalid, expired, consumed, or otherwise disallowed QR tokens; prevent duplicate active check-ins.
- Preserve closed sessions for historical review.
- Support location validation when real geofence enforcement is implemented and tested.

**Location verification must be represented honestly.** If location checks are simulated, display this exact message:

> Demo Mode: Location verification simulated.

A simulated result must never be represented as a verified geofence pass.

### 3. Live Occupancy & Attendance History

- Calculate occupancy from valid active attendance sessions rather than an independently editable counter.
- Reflect check-ins and check-outs in the active occupancy count.
- Keep completed attendance records available for authorized historical views.
- Calculate laboratory hours from attendance timestamps.
- Exclude incomplete sessions from finalized totals or clearly mark them as provisional.

### 4. Course Attendance & Missing-Student Identification

- Maintain course, enrolment, and scheduled laboratory session relationships.
- Show course-wise attendance rosters to authorized users.
- Compare active enrolments with qualifying attendance for the relevant session.
- Identify students without a qualifying attendance record.

A missing record is not automatically proof of physical absence if attendance collection is incomplete.

### 5. Equipment Inventory, Bookings & Loans

- Maintain quantity-based equipment inventory by laboratory.
- Let permitted users submit requests with quantities, time slots, and intended use.
- Let authorized Coordinators approve or reject requests for their assigned laboratories.
- Validate available stock and overlapping reservations before approval.
- Record physical equipment handover and return separately from booking approval.
- Preserve borrowing history and reject duplicate issuance or invalid returns.

**Booking approval is not equipment issuance.** A loan is created only when authorized staff record the physical handover.

### 6. Reporting & Administrative Oversight

- Provide personal laboratory-hour summaries.
- Provide course-level, laboratory-level, and department-level information within each role's scope.
- Support authorized inventory and asset summaries.
- Keep administrative corrections and privileged operations traceable.
- Preserve original values when records are corrected.

Daily/monthly reporting, export enhancements, scheduled backups, overdue reminders, and advanced audit interfaces are P1 or later work unless explicitly implemented and tested.

## User Roles

| Role | Primary responsibilities | Access boundary |
|---|---|---|
| **Student** | Check in/out, view personal attendance and lab hours, view occupancy and equipment availability, request equipment, review borrowing history, maintain permitted project information | Own records and permitted student actions; cannot approve own bookings, issue equipment, or alter attendance |
| **Faculty** | Review assigned course rosters and sessions, identify missing attendance, access relevant academic usage information, make authorized attendance corrections | Assigned courses and authorized academic scope; no system-wide role management |
| **Coordinator / Lab In-Charge** | Monitor assigned labs, manage inventory, process bookings, record equipment issue/return, review operational attendance, produce lab-level reports | Explicitly assigned laboratories; no system-wide security or Super Admin permission changes by default |
| **Head of Department (HOD)** | Review departmental attendance, occupancy, lab hours, utilization, inventory summaries, projects, and authorized reports/audit information | Authorized department; supervisory and read-only by default |
| **Super Admin** | Manage users, roles, account status, laboratory configuration, geofence settings, access policies, and authorized system-wide operations | System-wide administrative scope; privileged actions must be validated and traceable |

Role names alone do not grant access to every resource. For example, a Coordinator's laboratory access comes from active laboratory assignments, and an HOD's visibility comes from the authorized department.

## Technology Stack

| Layer | Technology | Responsibility |
|---|---|---|
| Frontend | HTML | Application structure and pages |
| Styling | CSS / Tailwind CSS | Responsive layout and interface styling |
| Client logic | JavaScript | User interaction and frontend behavior |
| Backend | Google Apps Script | Application operations, validation, authorization, and Google Sheets access |
| Initial data store | Google Sheets | Persistent application records organized into schema-defined tabs |
| Authentication | Institutional Google identity | Identity verification through the selected deployment's supported mechanism |

The initial stack is intentionally lightweight and suited to a small demonstration workload. It should not be described as a production-scale relational database architecture.

## Architecture

```text
┌──────────────────────────────────────────────┐
│                 User Browser                 │
│          HTML · Tailwind CSS · JS            │
│       Role-specific responsive dashboards    │
└──────────────────────┬───────────────────────┘
                       │ Application requests
                       ▼
┌──────────────────────────────────────────────┐
│             Google Apps Script               │
│                                              │
│  Identity & role checks                      │
│  Resource-scope authorization                │
│  Input and state validation                  │
│  QR / attendance / booking operations        │
│  Duplicate and conflict prevention           │
│  Audit events for required operations        │
└──────────────────────┬───────────────────────┘
                       │ Validated reads/writes
                       ▼
┌──────────────────────────────────────────────┐
│                Google Sheets                 │
│      Schema-defined tabs and relationships   │
│  Users · Labs · Attendance · Assets · ...    │
└──────────────────────────────────────────────┘
```

Google Sheets does not provide native relational foreign-key enforcement or full relational transaction guarantees. Apps Script must validate references and use appropriate locking, revalidation, idempotency, and failure handling for critical operations. The implementation must not claim guarantees that have not been implemented and tested.

## Google Sheets Database

The documented database baseline contains **14 tabs**. Each tab uses the exact schema-defined header names; stable IDs are generated by the backend, and row numbers are not permanent identifiers.

| # | Tab | Purpose | Priority |
|---:|---|---|---|
| 1 | `Users` | User profiles, roles, account status, department association | P0 |
| 2 | `Departments` | Department boundaries for HOD access | P0 |
| 3 | `Labs` | Laboratory configuration, location, capacity, geofence settings | P0 |
| 4 | `LabAssignments` | Staff-to-laboratory assignments and operational scope | P0 |
| 5 | `Courses` | Course information and responsible faculty | P0 |
| 6 | `Enrollments` | Student-course relationships | P0 |
| 7 | `Schedules` | Scheduled course and laboratory sessions | P0 |
| 8 | `Attendance` | QR attendance sessions, visit details, entry/exit timestamps | P0 |
| 9 | `QRTokens` | QR token references, expiry, and consumption status | P0 |
| 10 | `Assets` | Quantity-based equipment inventory by laboratory | P0 |
| 11 | `Bookings` | Equipment booking requests and decisions | P0 |
| 12 | `AssetLoans` | Physical equipment issuance and returns | P0 |
| 13 | `Projects` | Student project information and outcomes | P1 |
| 14 | `AuditLogs` | Administrative changes, corrections, and privileged-operation history | P0 minimum logging; P1 advanced reporting |

`AuditLogs` requires minimum logging from the outset for mandatory corrections and privileged operations, even though advanced audit search and reporting are P1. `Projects` is P1 and must not delay mandatory attendance and equipment workflows.

### Data integrity principles

- Generate stable unique IDs in the backend; never use sheet row numbers as permanent identifiers.
- Validate every foreign-key reference before creating or updating records.
- Use only documented status values for each table.
- Calculate occupancy, lab hours, and equipment availability from authoritative records.
- Preserve historical attendance and equipment transactions when a user or resource becomes inactive.
- Record actor, timestamp, reason, affected record, and traceable old/new values for required corrections.
- Use consistent timestamps and the configured institutional time zone.
- Protect contact information and identifiable records according to role and resource scope.

## Core Workflow States

### Attendance

```text
NOT_CHECKED_IN
       │ Valid QR + required validation passes
       ▼
     ACTIVE
       │ Valid check-out
       ▼
   COMPLETED
```

Invalid/expired/consumed tokens, duplicate active check-ins, failed required geofence checks, and check-outs without an active session must be rejected. `NOT_CHECKED_IN` is a logical state, not a separate row in the `Attendance` tab.

### Equipment booking and loan

```text
Booking:
PENDING ──────► APPROVED
   │
   ├──────────► REJECTED
   └──────────► CANCELLED

Physical loan:
APPROVED booking / permitted issue request
                   │ Authorized staff records handover
                   ▼
                 ISSUED
                   │ Authorized staff records return
                   ▼
                 RETURNED
```

Booking status and loan status are stored separately. Approval does not automatically create an issued loan. Availability and overlapping reservations must be revalidated before approval or issuance.

### Administrative corrections

A correction is allowed only when the acting role has the required permission, the record falls within the actor's authorized scope, and any mandatory reason is supplied. The original and corrected values must remain traceable. HOD access is read-only by default; Super Admin overrides must be authorized and audited.

## MVP Scope & Delivery Priorities

### P0 — Must work for the initial demonstration

- Institutional authentication and role-based access for all five roles.
- User profiles and laboratory configuration.
- QR check-in/check-out, token validation, and honest disclosure of geofence demo mode when applicable.
- Live occupancy and historical attendance.
- Course rosters and missing-attendance identification.
- Equipment inventory, booking requests, approval/rejection, conflict prevention, physical issue, and return.
- Personal lab-hour calculations and authorized asset summaries.
- Backend validation, duplicate prevention, required audit traceability, and data persistence.

### P1 — After P0 acceptance tests pass

- Daily and monthly attendance reports.
- CSV and PDF report export.
- Overdue equipment notifications or reminders.
- Project timeline and outcome tracking.
- Advanced audit browsing, filtering, and reporting.
- Scheduled Google Sheets backups and backup verification.
- Additional reporting filters and department-level customization.

### Later — Outside the initial two-day MVP

- Advanced GPS-spoofing detection and impossible-travel analysis.
- Predictive analytics and advanced anomaly detection.
- Extensive automated reporting and notification scheduling.
- Large-scale multi-laboratory optimization.
- Comprehensive encryption-key rotation and disaster recovery automation.
- Complex integrations with external institutional systems.
- Production-scale optimization and load testing.

Basic authorization, input validation, data integrity, and required audit traceability are not optional just because advanced features are deferred.

## Getting Started

The following is the intended setup sequence. Update this section with the final repository paths, deployment URLs, and project-specific commands once implementation is complete.

### Prerequisites

- Access to an authorized institutional Google account.
- Permission to create or use the project's Google Sheet.
- Access to Google Apps Script.
- A modern desktop or mobile browser.
- The completed frontend source and Apps Script backend.

### 1. Clone the repository

```bash
git clone <repository-url>
cd <repository-folder>
```

### 2. Configure Google Sheets

1. Create or select the Google Spreadsheet used by the application.
2. Create the 14 schema-defined tabs listed in [Google Sheets Database](#google-sheets-database).
3. Use the exact column names and order specified in the database schema document.
4. Populate required reference records—such as departments, laboratories, users, courses, and assignments—using valid IDs and relationships.
5. Configure the spreadsheet and script to use the intended institutional time zone.

Do not treat a spreadsheet as ready merely because the tabs exist. Validate headers, permitted values, unique identifiers, references, and required relationships.

### 3. Configure Google Apps Script

1. Set up the backend functions that read and write the configured spreadsheet.
2. Verify identity using the selected deployment's supported authentication mechanism.
3. Enforce role, account-status, and resource-scope checks on every protected operation.
4. Validate inputs, state transitions, QR token use, inventory availability, and booking conflicts server-side.
5. Add appropriate locking and failure handling to critical operations.
6. Keep secrets and privileged configuration out of public frontend code.

### 4. Configure the frontend

1. Update the frontend's backend endpoint or integration settings to match the deployed Apps Script application.
2. Keep UI permissions consistent with the backend rules, while treating frontend visibility only as a usability measure.
3. Ensure responsive behavior on desktop and mobile browsers.
4. Clearly label simulated or deferred functionality.

### 5. Deploy and test

Deploy the application using the project's approved Google Apps Script configuration. Test the deployed application end-to-end with authorized test accounts for all five roles before declaring a workflow operational.

> **Important:** No deployment URL, script ID, spreadsheet ID, environment variable name, or runnable development command is specified here because those details depend on the completed implementation. Replace the placeholders and add exact commands when the repository structure is finalized.

## Planned Repository Structure

The structure below is a **placeholder**, included for documentation planning. It is not a claim about the current repository. Replace it with the real tree after the frontend and backend files have been organized.

```text
labweb/
├── README.md
├── index.html
├── pages/
│   ├── student/
│   ├── faculty/
│   ├── coordinator/
│   ├── hod/
│   └── super-admin/
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
├── components/
├── services/
│   ├── api/
│   ├── auth/
│   └── validation/
├── config/
├── docs/
│   ├── 01-mvp-scope-and-requirements.md
│   ├── 02-roles-and-permissions.md
│   ├── 03-user-flows-and-state-diagrams.md
│   └── 04-database-google-sheets-schema.md
├── apps-script/
│   ├── Code.gs
│   ├── auth/
│   ├── attendance/
│   ├── bookings/
│   ├── inventory/
│   ├── reports/
│   └── validation/
└── tests/
    ├── authorization/
    ├── attendance/
    ├── bookings/
    └── data-integrity/
```

The folder names and file locations above are illustrative. The final structure must reflect the files actually created and the deployment approach selected for Apps Script.

## Security and Data Integrity

Security is part of the workflow, not a later enhancement.

- **Server-side authorization:** Every protected backend operation checks the authenticated identity, account status, role, and relevant course, laboratory, or department scope.
- **Trusted identity:** Never trust a role, student ID, or staff ID supplied by the browser without server-side verification.
- **Institutional access:** Restrict authentication to verified accounts in the approved institutional domains.
- **Input validation:** Reject missing fields, invalid IDs, invalid states, impossible timestamps, invalid quantities, and broken references.
- **Duplicate prevention:** Prevent duplicate attendance transactions, QR token consumption, issuance, and returns.
- **Conflict prevention:** Revalidate stock and overlapping reservations when processing bookings and equipment issuance.
- **Historical preservation:** Do not silently overwrite closed attendance sessions or completed equipment transactions.
- **Auditability:** Trace required corrections and privileged changes with actor, timestamp, reason, affected record, and old/new values.
- **Data minimization:** Limit contact details and identifiable records to authorized functions.
- **Honest status reporting:** Mark simulated or deferred features clearly; do not claim untested capabilities as operational.

Google Sheets is the initial data store for a small demonstration workload. The project should not claim database transaction guarantees, encryption, or production-grade security controls unless the relevant implementation has been verified.

## Testing & Definition of Done

The MVP is ready for demonstration only after the deployed workflows pass their acceptance tests.

- [ ] Approved institutional users can sign in; unauthorized domains are rejected.
- [ ] All five roles receive the correct access and resource scope.
- [ ] Direct unauthorized backend requests are rejected.
- [ ] User profiles and laboratory configuration can be saved and retrieved by authorized users.
- [ ] A valid QR check-in creates exactly one active session.
- [ ] A valid check-out closes the active session.
- [ ] Invalid tokens, duplicate check-ins, and invalid check-outs are rejected.
- [ ] Geofence behavior is tested, or the documented demo-mode message is displayed.
- [ ] Occupancy reflects valid active sessions.
- [ ] Course rosters identify students without qualifying attendance records for the relevant session.
- [ ] Equipment inventory and booking validation work correctly.
- [ ] Conflicting reservations and requests exceeding available stock are rejected.
- [ ] Authorized physical issuance creates a loan; valid returns close it.
- [ ] Personal lab hours and authorized asset summaries are consistent with source records.
- [ ] Required corrections and privileged operations remain traceable.
- [ ] Core records persist across page refreshes and application restarts.
- [ ] All mandatory P0 acceptance tests pass in the deployed application.

A successful page load is not proof that the feature works. Testing must follow the full path from user action through backend validation to the stored result.

## Project Documentation

The implementation should follow these baseline documents:

1. **MVP Scope and Requirements** — priorities, functional requirements, acceptance criteria, and definition of done.
2. **Roles and Permissions Matrix** — role responsibilities, authorization boundaries, and correction rules.
3. **User Flows and State Diagrams** — expected workflows, state transitions, and rejected operations.
4. **Database / Google Sheets Schema** — tabs, columns, IDs, relationships, permitted values, and integrity rules.

Resolve changes to scope, permissions, attendance states, or equipment transactions in the relevant project documentation before changing implementation behavior.

## Contributing

Contributions should preserve the agreed MVP scope and documented permission model.

1. Review the relevant requirements and user-flow documents before changing a workflow.
2. Keep authorization and validation in the backend.
3. Preserve the schema's exact identifiers, status values, and relationships unless the documentation is revised.
4. Add or update acceptance tests for changed behavior.
5. Clearly identify incomplete, simulated, or deferred functionality.
6. Do not commit credentials, tokens, private spreadsheet data, or other secrets.

## License

No license has been specified in the project baseline. Add a `LICENSE` file and update this section once the project owners select the applicable license.

---

<div align="center">

**LabWeb**  
*Making laboratory operations more visible, accountable, and connected.*

Karunya Institute of Technology and Sciences

</div>
