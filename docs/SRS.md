# Software Requirements Specification

# 1. Introduction

## 1.1 Purpose

This document specifies the requirements for the District Clinic Appointment & Queue System, a web and USSD-style application that allows patients to book clinic appointments and receive live queue updates by SMS, and allows clinic staff to manage the daily patient list and review no-shows and waiting times.

It is written for the development team (Group 2), the course facilitator who assesses the work, and any reviewer who needs to judge whether the delivered system meets what was agreed. It serves as the agreed baseline against which the architecture, tests and final product are checked.

## 1.2 Scope

The system is named the District Clinic Appointment & Queue System.

**The system will:**

- Allow patients to book, cancel and reschedule appointments through a web interface or a USSD-style menu simulation
- Place checked-in patients into a live daily queue
- Send patients their queue position and updates by SMS through a simulated gateway
- Allow clinic staff to check in booked patients, register walk-ins, call the next patient and record no-shows
- Provide clinic managers with a dashboard of no-shows and waiting times
- Enforce role-based access and record an audit trail of access to patient records

**The system will not:**

- Hold clinical records, diagnoses, prescriptions or treatment notes
- Process payments or insurance claims
- Integrate with a real SMS or USSD provider, or with any national health information system
- Store any real patient data; all data used in development and demonstration is synthetic

**Benefits and objectives.** Patients currently wait without knowing how long the wait will be, which means they must remain physically present for the whole period. The system aims to reduce uncertainty for patients, reduce crowding in waiting areas, and give clinic management evidence about no-shows and waiting times on which to base scheduling decisions.

## 1.3 Definitions, acronyms and abbreviations

| Term | Meaning |
| --- | --- |
| **Act 843** | Ghana's Data Protection Act, 2012 (Act 843), which governs the handling of personal data including health data |
| **ADR** | Architecture Decision Record |
| **Audit entry** | A record of who accessed or changed a patient record, what was done, and when |
| **BDD** | Behaviour-Driven Development |
| **Booking reference** | A short code issued on booking that identifies an appointment, e.g. ADC-4817 |
| **Bounded context** | A boundary within which a domain model and its terms apply consistently |
| **Gherkin** | The Given/When/Then notation used to express acceptance criteria |
| **INVEST** | A checklist for well-formed user stories: Independent, Negotiable, Valuable, Estimable, Small, Testable |
| **ISO 25010** | ISO/IEC 25010:2023, the product quality model used for the non-functional requirements |
| **MoSCoW** | A prioritisation scheme: Must have, Should have, Could have, Won't have |
| **NFR** | Non-Functional Requirement |
| **No-show** | A patient who booked an appointment but did not attend |
| **Queue position** | A patient's numbered place in the daily waiting order after check-in |
| **SLO** | Service Level Objective |
| **SMS gateway** | The external service that sends text messages; simulated in this project |
| **USSD** | Unstructured Supplementary Service Data, a menu-driven phone interface; simulated in this project |
| **Walk-in** | A patient who attends without a prior appointment |
| **WSJF** | Weighted Shortest Job First, a prioritisation formula |

## 1.4 References

1. DCIT 415 Capstone Projects & Assignments Handbook, First Semester 2026/2027, University of Ghana. Project P5 specification and milestone M1 deliverables.
2. Data Protection Act, 2012 (Act 843), Republic of Ghana.
3. ISO/IEC 25010:2023, Systems and software engineering — Systems and software Quality Requirements and Evaluation (SQuaRE) — Product quality model.
4. Evans, E. *Domain-Driven Design* — bounded contexts and context mapping.

## 1.5 Overview

Section 2 describes the system in general terms: its context, its users, its constraints and the assumptions on which the requirements rest. Section 3 states the specific requirements: the external interfaces, the functional requirements expressed as user stories with Gherkin acceptance criteria, and the measurable non-functional requirements. Section 4 gives the prioritised backlog, and Section 5 the context map that shows how the system is divided internally. Section 6 records the nine rules that the acceptance criteria depend on, with the reasoning behind each. Section 7 traces every required feature to the requirements that cover it.

---

# 2. Overall Description

## 2.1 Product perspective

The system is a new, self-contained product. It does not replace an existing computerised system; most district clinics in scope currently manage appointments and queues on paper.

It depends on one external service, an SMS gateway used to notify patients. For this project that gateway is simulated, and the system communicates with it through an adapter so that the rest of the system does not depend on any particular provider's interface.

The system is divided internally into five bounded contexts: Appointments, Queue, Notifications, Identity and Access, and Reporting. Section 5 describes these and the relationships between them.

## 2.2 Product functions

At a high level, the system provides:

- **Appointment booking** — patients choose a department, day and time slot, by web or USSD-style menu, and receive a booking reference
- **Appointment cancellation and rescheduling** — patients release or move a slot using their booking reference
- **Check-in and queueing** — staff admit booked patients and walk-ins into the daily queue in order
- **Queue progression** — staff call the next patient; positions update for everyone behind them
- **No-show recording** — staff mark non-attending patients, releasing the slot and logging the event
- **Patient notification** — the system sends queue position and updates by SMS, with retries on failure
- **Management reporting** — managers view no-show rates and waiting times over a date range
- **Account and role administration** — administrators create staff accounts and assign roles
- **Audit** — the system records access to and changes of patient records, reviewable by administrators

## 2.3 User characteristics

| User class | Characteristics | Implications for the system |
| --- | --- | --- |
| **Patients** | The general public. Varying literacy and digital familiarity; many on basic phones and low-bandwidth connections; often paying per megabyte | Simple flows, minimal typing, a USSD-style route for non-smartphone users, lightweight pages, SMS rather than an app for updates |
| **Receptionists / front desk staff** | Trained clinic staff using the system continuously through the clinic session; under time pressure with patients waiting | Fast, keyboard-friendly check-in; forgiving error handling; minimal clicks per patient |
| **Nurses** | Clinical staff who call patients through for consultation | A single clear action to advance the queue; no administrative clutter |
| **Doctors / clinicians** | Clinical staff who need to see their daily list and patient flow | Read-oriented view of the day's list |
| **Clinic managers** | Responsible for staffing and scheduling; occasional users | Clear reporting over date ranges; no access to patient records or account management |
| **Clinic administrators** | Responsible for staff accounts and compliance; occasional users | Account and role management; audit log review |
| **District health office** | Oversight body interested in clinic performance | Exportable or reviewable performance figures (future scope) |

## 2.4 Constraints

| ID | Constraint |
| --- | --- |
| **C-01** | Only sandbox or simulated external services may be used. No real payment credentials, real SMS provider accounts or real personal data (course handbook rule) |
| **C-02** | Personal data handling must comply with Act 843, including purpose limitation, access control and auditability |
| **C-03** | The system must be containerised (Docker and docker-compose) and deployable through a CI/CD pipeline to a staging environment |
| **C-04** | The implementation must use a mainstream stack as permitted by the handbook |
| **C-05** | The team consists of seven student developers working part-time across a twelve-week semester; scope must fit that capacity |
| **C-06** | Every team member must be able to explain any submitted code or document; AI assistance must be disclosed per milestone |
| **C-07** | Patients' phone numbers are the only personal identifier collected for notification purposes; no clinical data is stored |

## 2.5 Assumptions and dependencies

| ID | Assumption or dependency |
| --- | --- |
| **A-01** | Clinics operate on weekdays within defined clinic hours (taken as 07:00 to 18:00, Monday to Friday) |
| **A-02** | Patients have access to a mobile phone capable of receiving SMS |
| **A-03** | Clinic staff have a device with a browser and network access during clinic hours |
| **A-04** | Network connectivity at the clinic may be intermittent and of low bandwidth |
| **A-05** | A simulated SMS gateway is available for development, testing and demonstration |
| **A-06** | Appointments are allocated in fixed-length slots per department |
| **A-07** | A patient may hold at most one open appointment per department at a time |

## 2.6 Stakeholders

| Stakeholder | Role and interest |
| --- | --- |
| Patients | Book appointments, see their queue position by SMS, avoid long waits |
| Receptionists / front desk staff | Register patients, manage the daily appointment list, handle walk-ins |
| Nurses | See who is waiting and in what order |
| Doctors / clinicians | See their daily list and patient flow |
| Clinic manager | Monitors no-shows and waiting times to plan staffing and scheduling. Sees reports only |
| Clinic administrator | Creates staff accounts, assigns roles and reviews the audit log |
| District health office / health authority | Interested in clinic performance and compliance with health regulations |
| Data protection regulator | Cares about Act 843 compliance for patient data |
| SMS / USSD provider (simulated) | External service the system depends on for notifications |
| Development team and lecturer | Build and assess the system |

---

# 3. Specific Requirements

## 3.1 External interface requirements

### 3.1.1 User interfaces

| Interface | Description |
| --- | --- |
| **Patient web interface** | Responsive pages for booking, cancelling and checking queue position. Must work at phone width, load quickly on slow connections (NFR-08), and use plain language |
| **Patient USSD-style interface** | A numbered menu simulation for booking without internet access. Each step must tolerate invalid input without losing earlier choices |
| **Staff web interface** | Check-in by booking reference, walk-in registration, the live queue list with call-next and no-show actions, and recent audit entries |
| **Manager web interface** | Date-range dashboard showing no-show counts and rates, average and peak waiting times, and SMS delivery statistics |
| **Administration interface** | Staff account creation, role assignment and audit log review |

All interfaces must meet the accessibility expectations in the quality requirements: usable form controls with labels, adequate colour contrast, and touch targets large enough for use on a phone.

### 3.1.2 Hardware interfaces

The system requires no special hardware. It runs on standard server infrastructure and is accessed through commodity client devices: basic mobile phones for SMS and USSD, and smartphones, tablets or desktop computers with a modern browser for the web interfaces.

### 3.1.3 Software interfaces

| Interface | Purpose | Notes |
| --- | --- | --- |
| **SMS gateway (simulated)** | Sending queue and appointment messages to patients | Accessed through an adapter (anti-corruption layer) so provider details do not leak into the domain. Must tolerate the gateway being unavailable (NFR-06) |
| **USSD session simulation** | Driving the menu-based booking flow | Stateless per step, with session state held by the system |
| **Relational database** | Persisting appointments, queue entries, accounts, roles and audit entries | Specific product to be fixed by an ADR in M2 |
| **Health check endpoint** | `/health`, used by monitoring to compute availability (NFR-01) | Required by milestone M4 |

### 3.1.4 Communications interfaces

The web interfaces are served over HTTPS. Internal communication between contexts is in-process or over HTTP within the deployment. Queue position changes are published as events to the Notifications context. Messages to the SMS gateway are sent over its published interface and retried on failure.

## 3.2 Functional requirements

Functional requirements are expressed as user stories in INVEST form, each with Gherkin acceptance criteria including at least one alternative or failure path. There are thirteen stories, against the handbook minimum of twelve.

The acceptance criteria below incorporate the QA review of the first draft. Scenarios name their test data, state expected values, and verify the underlying state change rather than only the message shown to the user.

### 3.2.1 Patient-facing requirements

#### US-01 Book an appointment on the web

As a patient, I want to book an appointment online for a clinic and time slot, so that I do not have to travel to the clinic just to get a slot.

```gherkin
Scenario: Patient books an available slot
  Given the clinic has an available appointment slot for the selected department on Thursday at 09:00
  And the patient has provided all required booking details
  When the patient selects the available slot and confirms the booking
  Then the appointment is saved successfully
  And a unique booking reference is generated and displayed to the patient
  And the booking reference can be used to retrieve the saved appointment

Scenario: Patient tries to book a slot that has already been taken
  Given an appointment slot is available
  And another patient successfully books that slot
  When a second patient attempts to book the same slot
  Then the system rejects the second booking
  And the second patient is not assigned the occupied slot
  And the system displays other available appointment slots
```

#### US-02 Book an appointment through a USSD-style menu

As a patient without a smartphone, I want to book through a simple phone menu, so that I can use the clinic service without internet access.

```gherkin
Scenario: Patient completes a booking through the USSD-style menu
  Given the patient has opened the USSD-style booking menu
  And an appointment slot is available for the selected clinic and department
  When the patient follows the menu steps and confirms the appointment
  Then the appointment is saved successfully
  And a confirmation SMS is sent to the patient's phone number
  And the SMS contains the appointment details and a unique booking reference

Scenario: Patient enters an invalid menu option
  Given the patient has selected a clinic and is on the appointment selection step
  And the menu displays options 1, 2 and 3
  When the patient enters option 5
  Then the menu displays an invalid option error message
  And the appointment selection step is displayed again
  And the previously selected clinic is retained
```

#### US-03 Cancel or reschedule an appointment

As a patient, I want to cancel or reschedule my appointment, so that the slot can be used by someone else if I cannot attend.

```gherkin
Scenario: Patient cancels an upcoming appointment
  Given a patient has a confirmed appointment scheduled for Thursday at 09:00
  And the appointment is more than two hours away
  And the patient has the correct booking reference
  When the patient submits a cancellation request using the booking reference
  Then the appointment status is updated to cancelled
  And the appointment slot becomes available for another patient to book
  And the patient receives a cancellation confirmation

Scenario: Patient tries to cancel with a wrong booking reference
  Given no appointment exists with booking reference "ADC-9999"
  And another patient's appointment is confirmed and unchanged
  When the patient submits a cancellation request using booking reference "ADC-9999"
  Then the system displays an appointment not found message
  And no appointment is cancelled or modified
  And the other patient's appointment remains confirmed

Scenario: Patient successfully reschedules an appointment
  Given a patient has a confirmed appointment for Thursday at 09:00
  And another appointment slot is available for Friday at 10:00
  When the patient reschedules using the booking reference
  And selects Friday at 10:00
  Then the original appointment is updated
  And the Friday at 10:00 slot is assigned to the patient
  And the original Thursday at 09:00 slot becomes available
  And the patient receives a rescheduling confirmation
```

The booking reference is unchanged by a reschedule (D-07).

#### US-04 Receive my queue position by SMS

As a patient who has arrived at the clinic, I want to receive my queue position by SMS, so that I know how long I will wait.

```gherkin
Scenario: Patient is checked in and gets a queue position
  Given the patient has a confirmed appointment for today
  And the patient has a registered phone number
  When the receptionist checks the patient in
  Then the patient is added to today's queue
  And the patient receives an SMS showing their queue position

Scenario: SMS cannot be delivered
  Given the simulated SMS gateway is unavailable
  When a queue position message is due
  Then the queue operation succeeds
  And the system retries sending the message at least 3 times within 10 minutes
  And the failed delivery is recorded
```

The retry numbers come from NFR-06. Failed deliveries are recorded in the notification log, not the audit log (D-06).

#### US-05 Get updates as my turn approaches

As a patient waiting for a consultation, I want updates when my position changes or my turn is near, so that I can be ready without staying in the waiting room.

```gherkin
Scenario: Patient is notified that their turn is near
  Given the patient is third in today's queue
  When the two patients ahead of them are called in
  Then the patient receives an SMS notification
  And the SMS informs the patient that their turn is near

Scenario: A patient ahead leaves the queue
  Given the patient is fifth in today's queue
  When a patient ahead of them is marked as a no-show
  Then the patient's queue position is updated to fourth
  And the patient receives an SMS showing the new position
```

#### US-06 Check my queue position on demand

As a patient, I want to check my current queue position using my booking reference, so that I can see it without waiting for a message.

```gherkin
Scenario: Patient checks their position
  Given the patient is third in today's queue
  And the patient has a valid booking reference
  When the patient enters their booking reference
  Then the system displays their current queue position as 3
  And displays the estimated waiting time

Scenario: Booking reference does not belong to today's queue
  Given the booking reference belongs to a confirmed future appointment
  When the patient enters the booking reference to check their queue position
  Then the system informs the patient that the queue has not started for that appointment
  And the future appointment remains unchanged
  And the patient is not added to today's queue
```

The estimated waiting time is calculated as set out in D-03.

### 3.2.2 Staff-facing requirements

#### US-07 Register a walk-in patient

As a receptionist, I want to register a patient who arrives without an appointment, so that they join today's queue fairly.

```gherkin
Scenario: Receptionist registers a walk-in
  Given the receptionist is logged in
  And there are 3 patients waiting in today's queue
  When the receptionist enters a walk-in patient's name and phone number
  Then the patient is added as the fourth patient in today's queue
  And an SMS showing position 4 is sent to the patient's phone number

Scenario: Required details are missing
  Given the receptionist is registering a walk-in
  When they submit the form without a phone number
  Then the system displays a message that the phone number is required
  And the patient is not added to today's queue
```

A missing name is not covered here; a separate scenario would be needed if the team wants it tested.

#### US-08 Check in a booked patient

As a receptionist, I want to check in a patient who has a booking, so that they enter the queue in the right order.

```gherkin
Scenario: Receptionist checks in a patient with a valid booking
  Given the patient has an appointment today at 09:00
  And the patient has a valid booking reference
  When the receptionist enters the booking reference
  Then the patient is marked as arrived
  And the patient is placed in the queue according to their appointment time

Scenario: Checked-in patients are queued according to appointment time
  Given Patient A has a confirmed appointment at 09:00
  And Patient B has a confirmed appointment at 10:00
  And Patient B checks in before Patient A
  When Patient A checks in
  Then Patient A is placed first in the queue
  And Patient B is placed second in the queue

Scenario: Booking reference belongs to another day
  Given the booking reference belongs to an appointment on another day
  When the receptionist tries to check the patient in
  Then the system shows that no appointment was found for today
  And the patient is not added to today's queue
  And the system offers to register the patient as a walk-in
```

The second scenario exists because ordering by appointment time rather than arrival order is only provable with two patients. Queue ordering overall follows D-09.

#### US-09 Call the next patient

As a nurse, I want to call the next patient in the queue, so that patients are seen in order and everyone's position updates.

```gherkin
Scenario: Nurse calls the next patient
  Given 3 patients are waiting in the queue
  When the nurse selects "Call next"
  Then the first patient is marked as in consultation
  And the remaining waiting patients' positions are recalculated
  And patients who are no longer waiting are not assigned waiting positions

Scenario: Queue is empty
  Given no patients are waiting in the queue
  When the nurse selects "Call next"
  Then the system displays a message that the queue is empty
  And no patient status or queue position changes
```

"Positions are recalculated" replaces "reduced by one", which would otherwise have applied to patients already in consultation, completed or marked no-show.

#### US-10 Mark a patient as a no-show

As a receptionist, I want to mark a patient who did not arrive as a no-show, so that the queue moves on and the clinic can track missed appointments.

```gherkin
Scenario: Receptionist marks an eligible appointment as a no-show
  Given a patient has a confirmed appointment
  And the patient has not checked in
  And the allowed waiting time for the appointment has passed
  When the receptionist marks the appointment as a no-show
  Then the appointment status is updated to no-show
  And the patient is removed from the active queue
  And the action is recorded in the audit log

Scenario: Receptionist cannot mark an already checked-in patient as a no-show
  Given a patient has already been checked in
  And the patient is in the active queue
  When the receptionist attempts to mark the patient as a no-show
  Then the system blocks the action
  And displays a message that the patient has already checked in
  And the patient's status remains unchanged
```

The allowed waiting time is set by D-01; what marking a no-show does is set by D-02; Undo follows D-08.

#### US-11 View no-shows and waiting times

As a clinic manager, I want a dashboard of no-shows and waiting times, so that I can plan staffing and reduce missed appointments.

```gherkin
Scenario: Manager views dashboard statistics for a selected date range
  Given the manager is logged in
  And appointment and queue records exist for the selected date range
  When the manager selects the date range
  Then the dashboard displays the total number of no-shows
  And displays the average waiting time
  And displays the no-show rate
  And the displayed statistics match the underlying records

Scenario: Manager views dashboard for a date range with no appointments
  Given the manager is logged in
  And no appointment records exist for the selected date range
  When the manager opens the dashboard for that date range
  Then the system displays a message indicating that no data is available
  And no appointment statistics are displayed for that range
  And no misleading charts are displayed
```

The dashboard metrics and the no-show-rate formula are set by D-04.

#### US-12 Manage staff accounts and roles

As a clinic administrator, I want to create staff accounts and assign roles, so that each person can only see and do what their job requires.

```gherkin
Scenario: Administrator creates a staff account with a role
  Given the clinic administrator is logged in
  When the administrator creates a staff account with valid details
  And assigns the Receptionist role to the account
  Then the staff account is created successfully
  And the new receptionist can log in
  And the receptionist can access permitted receptionist functions
  And the receptionist cannot access functions restricted to other roles

Scenario: Receptionist is denied access to staff account management
  Given a receptionist is logged in
  When the receptionist attempts to open the staff account management page
  Then the system denies access
  And displays an access-denied message
  And records the unauthorized access attempt in the audit log
```

Which functions each role may access is set by the matrix in D-05. These two scenarios are not sufficient on their own to satisfy NFR-04, which requires automated authorisation tests across every role-restricted endpoint.

#### US-13 Review the audit log

As a clinic administrator, I want to review who accessed or changed patient records, so that we can show Act 843 compliance and investigate misuse.

```gherkin
Scenario: Administrator reviews patient record activity
  Given a patient record has been viewed and updated by a staff member
  And the administrator is logged in
  When the administrator opens the audit log
  Then the audit log displays an entry for viewing the patient record
  And displays an entry for updating the patient record
  And each entry shows the staff member who performed the action
  And each entry shows the action performed
  And each entry identifies the affected patient record
  And each entry shows the date and time of the action

Scenario: Nurse is denied access to the audit log
  Given a nurse is logged in
  When the nurse attempts to open the audit log
  Then the system denies access
  And displays an access-denied message
  And the nurse cannot view any audit-log entries
```

The one-second audit-entry requirement in NFR-05 is verified as a separate timing check, not in these scenarios.

## 3.3 Non-functional requirements

Each requirement names an ISO 25010 quality characteristic, a metric and threshold, the conditions under which it applies, and how it will be verified. The thresholds below are the team's proposed targets and are subject to confirmation at review.

| ID | ISO 25010 quality | Requirement | Verification |
| --- | --- | --- | --- |
| **NFR-01** | Reliability (availability) | Booking, check-in and queue functions are available at least 99.5% of the time during clinic hours (07:00 to 18:00, Monday to Friday), measured per calendar month. | Uptime monitoring on staging using the `/health` endpoint; availability calculated from the monitoring record. |
| **NFR-02** | Performance efficiency (time behaviour) | 95% of check-in and call-next requests complete within 2 seconds with 20 concurrent staff users. | Load test run against staging; result recorded in the test report. |
| **NFR-03** | Performance efficiency (timeliness) | 95% of queue-position messages are handed to the SMS gateway (simulated) within 30 seconds of the queue change. | Timestamps in the audit log compared in an integration test using the simulated gateway. |
| **NFR-04** | Security (access control) | Every staff endpoint rejects users without the required role with an access-denied response; 100% of role-restricted endpoints are covered by automated authorisation tests. | Authorisation tests in the CI pipeline; the build fails if an endpoint has no test. |
| **NFR-05** | Security (accountability, Act 843) | Every view or change of a patient record creates an audit entry (who, what, which record, when) within 1 second; 100% of record endpoints are covered. | Automated tests that call each record endpoint and check the audit entry. |
| **NFR-06** | Reliability (fault tolerance) | When the SMS gateway is unavailable, 100% of queue operations still succeed, and undelivered messages are retried at least 3 times within 10 minutes. | Integration test that makes the simulated gateway fail on purpose. |
| **NFR-07** | Interaction capability (usability) | At least 90% of first-time receptionists can register a walk-in in under 60 seconds after a 5-minute orientation, tested with at least 5 users. | Usability test with classmates acting as receptionists; results recorded. |
| **NFR-08** | Performance efficiency (low bandwidth) | The patient booking page loads in under 3 seconds on a throttled slow-3G connection, with total page size under 500 KB. | Browser developer-tools throttling or a Lighthouse run on staging. |
| **NFR-09** | Maintainability (testability) | Domain code (appointments and queue logic) has at least 70% branch coverage, enforced as a gate in CI. | Coverage report produced by the CI pipeline. |

---

# 4. Prioritised Backlog

**WSJF** = (business value + time criticality + risk reduction or opportunity enablement) ÷ job size. Scores use a relative scale of 1, 2, 3, 5, 8, 13, and are the team's proposed estimates. Stories are listed in WSJF order.

| Story | Business value | Time criticality | Risk reduction | Job size | WSJF | MoSCoW |
| --- | --- | --- | --- | --- | --- | --- |
| US-08 Check in a booked patient | 8 | 8 | 5 | 3 | (8+8+5)/3 = 7.0 | Must |
| US-09 Call the next patient | 8 | 8 | 5 | 3 | (8+8+5)/3 = 7.0 | Must |
| US-12 Manage staff accounts and roles | 8 | 8 | 13 | 5 | (8+8+13)/5 = 5.8 | Must |
| US-07 Register a walk-in patient | 8 | 5 | 3 | 3 | (8+5+3)/3 = 5.3 | Must |
| US-01 Book an appointment on the web | 13 | 8 | 5 | 5 | (13+8+5)/5 = 5.2 | Must |
| US-03 Cancel or reschedule | 5 | 3 | 3 | 3 | (5+3+3)/3 = 3.7 | Should |
| US-10 Mark a no-show | 5 | 3 | 3 | 3 | (5+3+3)/3 = 3.7 | Must |
| US-04 Queue position by SMS | 8 | 5 | 5 | 5 | (8+5+5)/5 = 3.6 | Must |
| US-13 Review the audit log | 5 | 3 | 8 | 5 | (5+3+8)/5 = 3.2 | Should |
| US-06 Check position on demand | 3 | 2 | 2 | 3 | (3+2+2)/3 = 2.3 | Could |
| US-05 Updates as turn approaches | 5 | 3 | 3 | 5 | (5+3+3)/5 = 2.2 | Should |
| US-02 Book through USSD-style menu | 5 | 3 | 3 | 8 | (5+3+3)/8 = 1.4 | Could |
| US-11 View no-shows and waiting times | 5 | 2 | 3 | 8 | (5+2+3)/8 = 1.3 | Must |

**Why some MoSCoW ratings differ from the WSJF order**

- US-11 is a Must even though its WSJF is lowest, because the handbook lists the staff dashboard (no-shows and waiting times) as a minimum feature.
- US-04 is a Must because live SMS queue updates are a minimum feature.
- US-02 is a Could because the handbook allows booking by "web/USSD-style simulation", so the web booking in US-01 satisfies the requirement on its own.
- US-12 is a Must because role-based access is a named engineering challenge for this project.
- US-13 is a Should for now. Writing audit entries is covered by NFR-05; reviewing them on screen can follow.

**Won't have this semester:** a real SMS or USSD provider, and any real patient data (the handbook permits sandbox or simulated services only).

---

# 5. Context Map

The system is divided into five bounded contexts.

```mermaid
flowchart LR
  ID["Identity and Access<br/>staff accounts, roles, audit log"]
  AP["Appointments<br/>booking, cancelling, slots"]
  QU["Queue<br/>check-in, positions, call next, no-shows"]
  NO["Notifications<br/>SMS and USSD messages"]
  RE["Reporting<br/>dashboard"]
  EX(("Simulated SMS and<br/>USSD provider"))

  AP -->|"customer/supplier: arrivals and bookings"| QU
  QU -->|"events: position changed"| NO
  NO -->|"anti-corruption layer"| EX
  AP -->|"conformist: booking data"| RE
  QU -->|"conformist: waiting and no-show data"| RE
  ID -.->|"open host service: roles and audit"| AP
  ID -.->|"open host service: roles and audit"| QU
  ID -.->|"open host service: roles and audit"| RE
```

| Context | Responsibility | Relationships |
| --- | --- | --- |
| **Appointments** | Slots, bookings, cancellations and rescheduling | Supplies booking and check-in data to Queue; feeds Reporting |
| **Queue** | Check-in, positions, calling the next patient, no-shows | Receives bookings from Appointments; publishes position changes to Notifications; feeds Reporting |
| **Notifications** | Builds and sends SMS and USSD-style messages, with retries | Listens to Queue events; talks to the external provider through an adapter so provider details stay out of the other contexts |
| **Identity and Access** | Staff accounts, roles and the audit log | Provides roles and audit recording to all other contexts |
| **Reporting** | Dashboard of no-shows and waiting times | Reads data from Appointments and Queue without changing it |

---

# 6. Agreed Rules and Decisions

The QA review of the first draft left nine rules undefined. Each is decided below with its reasoning. These are the rules the acceptance criteria in section 3.2 and the tests in milestone M6 are written against.

## D-01 Grace period before a no-show

A patient may be marked as a no-show 30 minutes after their scheduled appointment time. The timer starts at the appointment time, not at the start of the clinic session. The threshold is configurable per department.

Patients travel by shared transport and traffic is unpredictable, so a short grace period would penalise people for circumstances outside their control. Thirty minutes is forgiving without allowing one absence to stall the queue.

## D-02 What marking a no-show does

The appointment status becomes no-show, the patient is removed from the active queue, and the slot is released so that it counts as free capacity for the remainder of that day.

Both effects are required. Removing the patient keeps the queue accurate; releasing the slot means the day's capacity figures and any same-day reallocation reflect reality.

## D-03 Calculating the estimated wait

Estimated wait = (number of patients ahead) × (rolling mean consultation time for that department today). Where fewer than five consultations have completed, a configured default of 12 minutes per consultation is used instead. The result is rounded to the nearest 5 minutes and presented as "about N minutes".

A rolling same-day mean adapts to how the clinic is actually running. Rounding matters: an estimate presented as "34 minutes" claims a precision the method does not have.

## D-04 Dashboard metrics and the no-show rate

The dashboard shows total appointments split into booked and walk-in, no-show count, no-show rate, average waiting time, the longest day in the range, and SMS delivery rate.

No-show rate = no-shows ÷ booked appointments. Walk-ins are excluded from the denominator, because a walk-in cannot fail to attend an appointment that was never made. Including them understates the rate.

## D-05 Role-permission matrix

This matrix is what the authorisation tests required by NFR-04 are written against.

| Function | Receptionist | Nurse | Manager | Administrator |
| --- | --- | --- | --- | --- |
| View today's queue | Yes | Yes | Read-only | No |
| Check in a booked patient | Yes | No | No | No |
| Register a walk-in | Yes | No | No | No |
| Call the next patient | No | Yes | No | No |
| Mark a consultation complete | No | Yes | No | No |
| Mark a no-show / undo | Yes | No | No | No |
| View a patient record | Yes | Yes | No | No |
| View reports | No | No | Yes | No |
| Create staff accounts, assign roles | No | No | No | Yes |
| View the audit log | No | No | No | Yes |

The manager cannot open patient records, because planning staffing does not require naming a patient. The administrator cannot either, which means the person who reads the audit log is not someone the audit log is likely to concern.

## D-06 Failed SMS records

Failed deliveries are written to a separate notification log, not the audit log. Each record holds the message id, recipient number, message type, the queue event that triggered it, attempt number, a timestamp per attempt, the gateway error returned, and the final state.

The audit log exists to answer who accessed patient data, which is an Act 843 question. Filling it with gateway timeouts makes that harder to review, and the two have different retention needs.

## D-07 Reschedule and the booking reference

Rescheduling keeps the original booking reference. The appointment record holds a history of its changes.

The patient has the reference written down or held in an SMS. Issuing a new one puts two references in circulation and creates a support call when the patient arrives quoting the old one.

## D-08 Undo on a no-show

The patient is restored to the queue by the ordering rule in D-09, as if they had just checked in. A new audit entry records the undo; the original no-show entry is never removed.

"Original position" is undefined once other patients have been called. Re-applying the normal rule is one behaviour instead of two, and a patient wrongly marked absent keeps the place their appointment time earns them.

## D-09 Queue ordering and walk-ins

Every queue entry sorts on a single key. For a booked patient it is their appointment time; for a walk-in it is the time they registered. The queue is that list in ascending order.

This one rule produces the behaviour both US-07 and US-08 describe. A walk-in joins behind everyone whose appointment time has already passed. A booked patient who checks in late still sits ahead of one with a later appointment. A walk-in arriving at 09:15 goes ahead of a 10:00 booking, which is correct, since nobody should be seen before their own appointment time.

## Further test coverage required

- US-08: a booking reference that does not exist at all, as distinct from one belonging to another day.
- US-08: the same booking reference entered twice, which must not create duplicate queue entries.
- US-10: the Undo action, against the rule in D-08.
- NFR-04: automated authorisation tests across every cell of the D-05 matrix, not only the two scenarios in US-12.
- NFR-05: a timing check that audit entries are written within one second.

---

# 7. Traceability

Each minimum feature required by the project specification is covered by at least one user story and one non-functional requirement.

| Required feature (P5) | Covered by stories | Supported by NFRs |
| --- | --- | --- |
| Booking via web / USSD-style simulation | US-01, US-02, US-03 | NFR-01, NFR-08 |
| Live queue position updates by SMS | US-04, US-05, US-06 | NFR-03, NFR-06 |
| Staff dashboard: no-shows, waiting times | US-11 | NFR-01, NFR-02 |
| Health data privacy (Act 843) | US-12, US-13 | NFR-04, NFR-05 |
| Availability during clinic hours | US-08, US-09 | NFR-01, NFR-02, NFR-06 |
| Role-based access | US-12 | NFR-04 |
| Audit trails | US-10, US-13 | NFR-05 |
