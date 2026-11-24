# District Clinic Appointment & Queue System

![CI](https://github.com/pkay025/team-2-clinic-queue/actions/workflows/ci.yml/badge.svg)

DCIT 415 Advanced Software Engineering Capstone Project, University of Ghana (First Semester 2026/2027).

## What it is about?

A system for district clinics where patients book appointments, receive live queue position updates by SMS, and clinic staff manage the daily list and see no-shows and waiting times.

**Minimum features**

- Booking through a web interface or a USSD-style simulation
- Live queue position updates by SMS (simulated)
- Staff dashboard showing no-shows and waiting times

**Engineering focus**

- Health data privacy (Act 843)
- Availability during clinic hours
- Role-based access
- Audit trails

This project uses sandbox or simulated services only. No real patient data is stored anywhere.

## Staging

Staging URL: _to be added (M5)_

## How to run

_To be completed once the tech stack is chosen (M4)._

```bash
# 1. Clone the repository
git clone <repo-url>
cd <repo>

# 2. Start the application
docker-compose up --build
```

## Team

| Name | Role | GitHub |
| --- | --- | --- |
| Emmanuel Nunoo Mensah | Product Owner / Team Lead | pkay025 |
| Emmanuel Kwaku Duah Cobbinah | Architect / Tech Lead | emmanuelcobbinah007 |
| Felicity Dzidzorli Zanu - Tsikata | QA Lead | Fel-Tsikata |
| Kelvin Kwame Tetteh | DevOps / Platform Lead | kelvintk1 |
| ⁠Purity Abena Kyei | Security & Privacy Lead | Purity1919 |
| Ama Okyerewaa Ofori and Bernardina Nyanyuie Senadza | UX & Documentation Lead | Oky-of and D-codes-it |

## Repository structure

```
README.md
docs/
  requirements.md      # M1
  architecture/        # C4 diagrams, quality scenarios (M2)
  adr/                 # Architecture decision records
  TEST_STRATEGY.md     # M6
  THREAT_MODEL.md      # M7
  PLAN.md              # M8
  ai-use.md            # AI-use statements, one section per milestone
src/                   # Application code
tests/                 # unit/, integration/, features/ (BDD), e2e/
Dockerfile
docker-compose.yml
.github/workflows/ci.yml
```

## How we work

- Every task is a GitHub issue with one owner.
- No direct pushes to `main`: work on a branch and open a pull request.
- Every pull request needs at least one review before merging.
- Sprints run 1 to 2 weeks, with a short weekly status note.

## Milestones

| Milestone | Due | Status |
| --- | --- | --- |
| M1 Requirements | Week 4 | Completed |
| M2 Architecture | Week 6 | In Progress |
| M3 Code quality | Week 7 | Not started |
| M4 Containerised prototype | Week 8 | Not started |
| M5 CI/CD | Week 9 | Not started |
| M6 Testing | Week 10 | Not started |
| M7 Security & privacy | Week 11 | Not started |
| M8 Plan & risk | Week 11 | Not started |
| Final product and report | Week 12 | Not started |
