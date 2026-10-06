# Mzima | Proposed System Design

## Capacity-aware queue management for outpatient triage

**Draft 1 | 6 October 2026 | Outpatient departments of Level 6 public hospitals, Kenya**

**Status:** concept stage. Mzima is a proposed system. It has not been built, piloted or tested with real patients. Examples in this document use illustrative data, and every target is a hypothesis to be confirmed in the pilot.

## How this document is organised

| Part | Sections | Question it answers |
|---|---|---|
| 1 Understand | 1 to 2 | What problem are we solving, and for whom? |
| 2 How it works | 3 to 4 | What happens to a patient, and how is the queue ordered? |
| 3 How it is built | 5 to 8 | Which AI, architecture and technology will we use, and how is data protected? |
| 4 How we deliver | 9 to 12 | How will we build, pilot, measure and manage risk, and what happens next? |

# PART 1 | UNDERSTAND

## 1 The problem and our solution

Outpatient queues in public hospitals, especially Level 6 (national referral) facilities, are mostly managed by arrival order. Patient arrivals surge and triage staffing changes through the day, but the queue does not respond. Patients pile up faster than nurses can assess them, while capacity elsewhere in the hospital may sit idle. Critically ill patients can wait behind minor cases.

### What Mzima will do

- Order the queue so patients with red flags are found and seen early.
- Measure capacity live (open triage stations, nurses, triage times) and forecast arrivals for the next six hours.
- Recommend actions to the shift lead, such as opening a station or diverting low-acuity patients, before the queue becomes a crowd.
- Goal: a shorter and more predictable time to first assessment.

### Why the African context shapes the design

Public hospitals in Kenya and across Africa face high walk-in volumes, limited and unevenly distributed staff, demand that swings through the day, and a mix of digital and manual workflows. Mzima is therefore designed to work offline, to run beside paper processes, to need no patient ID, and to support English and Kiswahili.

## 2 Who will use Mzima

| Role | What they need | Mzima screen |
|---|---|---|
| Registration clerk | Check patients in quickly and run the door screen | Check-in screen |
| Triage nurse | Know who to call next and flag deterioration | Nurse queue screen |
| Shift lead | Match staff to demand and act on alerts | Live board |
| OPD manager | See trends, plan rosters and report | Analytics dashboard |
| ICT or records officer | Manage users, stations and data export | Admin console |
| Patient | Know the order and expected wait | Waiting-room display, optional SMS |

# PART 2 | HOW IT WORKS

## 3 Step-by-step patient workflow

Mzima targets the time from arrival to first assessment. Each patient moves through six steps, with one continuous loop running beside them.

| Step | Mzima system step | Staff step | What happens | Target |
|---|---|---|---|---|
| 1 | Check in | Clerk | Issue a numbered token (no ID needed). Record age band and visit type: new, review or refill. | Under 1 min |
| 2 | Door screen | Clerk | Ask yes/no red-flag questions: breathing difficulty, convulsion, unconscious, heavy bleeding, severe pain, pregnancy with bleeding or pain, infant under 2 months. Any yes sets fast-track. | 30 seconds |
| 3 | Rank queue | Mzima | Fast-track goes to the top and alerts the nearest nurse. All others are ordered by waiting time. | Automatic |
| 4 | Call next | Nurse | The nurse sees the next token with its flags and may call any token. An override takes one tap and a reason. | Fast-track within 5 min |
| 5 | Clinical triage | Nurse | Vitals and a category on the hospital's scale, such as SATS or TEWS (ETAT for children). First-assessment time is recorded. | Hospital protocol |
| 6 | Route and close | Nurse | Emergency bay, consultation clinic, fast stream (reviews, refills), lab, pharmacy or referral. | One tap |
| Loop | Monitor and respond | Mzima and shift lead | Live wait, forecast and alerts feed the shift lead, who acts. | Continuous |

### Runs throughout: Monitor and respond

Mzima forecasts arrivals, estimates the waiting time and alerts the shift lead, who opens stations, moves staff or diverts low-acuity patients to the fast stream.

### Deterioration while waiting

Any staff member can raise a red flag on a waiting token in one tap. It moves to fast-track at once and the nearest nurse is alerted.

## 4 How the queue will be ordered

Patients have no clinical category until a nurse sees them, so the queue before triage is ordered with a short door screen and a simple, visible score.

| Rule | How it will work |
|---|---|
| Fast-track | Any door-screen red flag goes above all other tokens. The nearest nurse is alerted, and the shift lead if no one responds in 3 min. Target: assessed within 5 min. |
| Standard order | Score = minutes waited × weight. The highest score is called first. Weight is 1.0, or 1.5 for under 5, over 65 and pregnant patients. With equal weights this is plain arrival order, so staff can understand and trust it. |
| Wait cap | A token waiting over 120 min is promoted and the shift lead is alerted, so no one is left behind. |
| Fast stream | Review, refill and procedure-only visits are offered the fast stream when a service point is free. This uses idle capacity outside triage. |
| Override | A nurse can call any token. A reason code is logged. Clinical judgement always comes first. |
| Settings | Weights, caps and thresholds are set per hospital and approved by the clinical lead. |

# PART 3 | HOW IT IS BUILT

## 5 The AI inside Mzima

Mzima uses AI where prediction helps, and plain rules where clinicians need to understand the logic. AI will never make a clinical decision.

| Component | What it does | Method | Python tools |
|---|---|---|---|
| 1 Arrival forecast | Predicts patients arriving each hour for the next 6 hours | Seasonal average first, then gradient boosting using hour, weekday, month, holidays and clinic days | pandas, statsmodels, scikit-learn |
| 2 Wait-time estimate | Estimates the wait for the next arrival | Queueing model (M/M/c) from the live queue and open stations, with a learned correction from past waits | NumPy, SciPy, scikit-learn |
| 3 Recommender | Turns forecast and live wait into suggested actions | Threshold rules agreed with the clinical lead, checked against the forecast | Python, FastAPI |
| 4 Queue ranking | Orders waiting patients | Transparent score and red-flag fast-track (rules, not machine learning) | Python |
| 5 Simulator | Compares first-come-first-served with Mzima on simulated demand before any pilot | Discrete-event simulation | SimPy, NumPy |

### Cold start

Until the hospital has 8 to 12 weeks of check-in data, the forecast will start from seasonal averages, OPD register counts and simulated data, and improve as real data arrives.

## 5.1 Example output

**Illustrative data:** forecast arrivals against triage capacity.

- 3 stations: 36/hr
- 5 stations: 60/hr
- 07:00 — 24 patients
- 08:00 — 41 patients
- 09:00 — 55 patients
- 10:00 — 58 patients
- 11:00 — 47 patients
- 12:00 — 33 patients

Teal bars are within 3-station capacity and amber bars are above it. Dashed lines show capacity with 3 and 5 stations. Illustrative data, assuming 5 minutes per triage (12 patients per nurse-hour). Recommendation shown to the shift lead: open 2 more stations from 08:30 to 11:30.

### Responsible AI

- **Human in charge.** Every recommendation can be accepted, dismissed or overridden, and the reason is logged.
- **Explainable.** Each recommendation shows why, for example: forecast 58 per hour against capacity of 36.
- **Monitored.** Forecast error is tracked weekly and the model is retrained monthly.
- **Fairness checks.** Waiting times are compared across age band, sex and visit type.
- **Minimal data.** Models use counts and timestamps, not names or diagnoses.

## 6 System architecture

Mzima will be offline-first. A server inside the hospital runs everything needed for the queue, so a lost internet connection never stops triage. The cloud is used for sync, backup and reporting.

### Devices at the point of care

Web app in the browser: HTML, CSS, JavaScript, React.

- Check-in tablet
- Triage station tablet
- Shift lead board (PC or tablet)
- Waiting-room display
- Paper token fallback

Hospital network; no internet needed.

### Hospital server (on premise)

Python, FastAPI, scikit-learn, PostgreSQL.

Components:

- Queue engine
- Forecast model
- Wait-time estimate
- Recommender
- Audit log
- PostgreSQL database

Encrypted sync when online is added after the pilot starts.

### Cloud

Kenya-based region.

- Analytics and reports
- Model retraining and backup

### Later (Phase 2): integrations

- HMIS or EMR via FHIR
- SMS gateway and DHIS2

## 7 Technology stack

Languages: Python for the back end and AI, JavaScript, HTML and CSS for the screens, and SQL for the database.

| Layer | Technology | Used for |
|---|---|---|
| Front end | HTML5, CSS3, JavaScript (ES2022), React with Vite, Tailwind CSS, Chart.js | Screens for clerks, nurses, shift leads and managers, and the dashboard charts |
| Offline mode | Progressive web app: service worker and IndexedDB | Screens keep working when the internet drops, and actions sync later |
| Back end | Python 3.11, FastAPI, Uvicorn, Pydantic, WebSockets | REST API for the queue, with live updates pushed to screens |
| Database | PostgreSQL with SQLAlchemy and Alembic | Reliable storage and tracked schema changes |
| AI and data | pandas, NumPy, SciPy, scikit-learn, statsmodels, SimPy | Forecasting, wait estimate and simulation |
| Scheduled jobs | APScheduler | Hourly forecasts and monthly retraining |
| Security | JWT login, role-based access, HTTPS, argon2 password hashing | Protecting access to patient data |
| Quality | Git and GitHub, pytest, ESLint, Prettier, GitHub Actions | Version control, tests and checks on every change |
| Deployment | Docker and Docker Compose on a mini-PC or hospital server | The same setup in every hospital |
| Integrations (later) | FHIR (fhir.resources), Africa's Talking SMS API, DHIS2 API | Connect to the hospital system, send SMS, report to national systems |

### Keep it simple

The prototype needs only the front end, back end, database, AI libraries and security rows. Offline mode, deployment and integrations come as the pilot approaches. If the team is new to React, the prototype screens can use plain HTML, CSS and JavaScript against the same API.

## 8 Data and privacy

| Data | What is stored |
|---|---|
| Visit | Token number, arrival time, age band, visit type, red flags, triage start and end time, triage category, destination, left unseen |
| Station session | Station, nurse sign-in and sign-out times |
| Queue event | Every call, override, flag and promotion, with who, when and the reason |
| Forecast and recommendation | Predicted and actual arrivals, suggested action, accepted or dismissed |
| Roster | Planned staff per shift |

- No names or diagnoses in the queue. A link to the hospital's patient ID is optional and comes later.
- Built to the Data Protection Act, 2019: impact assessment, notice at registration, and data hosted in Kenya.
- Role-based access, encryption in transit and at rest, and an audit log of overrides and data views.
- Waiting-room screens show token numbers only.

# PART 4 | HOW WE DELIVER

## 9 Build plan, step by step

| Step | What we build | Done when |
|---|---|---|
| 1 | Set up GitHub repository, Docker Compose with FastAPI, PostgreSQL and React, and coding rules | The app starts with one command |
| 2 | Simulated data: a Python script that generates realistic arrivals (busy mornings, weekday patterns) and triage times | Three months of simulated visits are in the database |
| 3 | Queue engine and API: check-in, door screen, ranking, call next and close-token endpoints, with tests | A visit runs end to end in tests |
| 4 | Screens: check-in, nurse queue, shift lead board and waiting-room display | A clerk and nurse can run a full visit |
| 5 | Forecast model: seasonal baseline, then gradient boosting, with an accuracy report | Hourly forecast error is measured on held-out data |
| 6 | Wait estimate and recommendations: M/M/c estimate, recommendation rules and status levels | The board shows live wait and suggested actions |
| 7 | Simulation study: SimPy comparison of first-come-first-served and Mzima | Results report, clearly labelled as simulated |
| 8 | Security and testing: roles, logging, privacy checks and a usability test with nurses | Checklist passed |
| 9 | Demo and pilot proposal: demo script, presentation and a one-page pilot proposal for a hospital | Ready to present |

### Suggested team split

Front end, back end, AI and data, and a clinical and partnerships lead who handles the hospital, ethics approval and nurse feedback.

## 10 Pilot plan and success metrics

After the prototype, Mzima will be tested in four phases. Weeks count from approval to start.

| Phase | Weeks | What happens | Exit condition |
|---|---|---|---|
| 0 Baseline | 1 to 2 | Record paper-token times and a time-and-motion sample. Agree thresholds. Obtain ethics and data approvals. | Baseline waits documented |
| 1 Shadow mode | 3 to 6 | Mzima runs beside the process and recommends to the shift lead only. Calling order is unchanged. | Hourly forecast within ±20% of actual |
| 2 Live pilot | 7 to 14 | Ranking and alerts go live in one OPD stream, with a weekly nurse review and safety check. | KPIs compared with baseline, no open safety concerns |
| 3 Evaluate and extend | 15 to 16 | Report results, adjust rules and plan a second site. | Go or no-go for scale-up |

### Success metrics

| Metric | Definition | Target (hypothesis) |
|---|---|---|
| Median time to first assessment | Check-in to start of triage | 30% lower than baseline |
| 90th percentile wait | Longest waits, which show who is left behind | 25% lower than baseline |
| Fast-track response | Share of red-flag patients assessed within 5 min | 95% or higher |
| Left without being seen | Tokens closed with no triage | 20% lower than baseline |
| Override rate | Share of calls where the nurse overrode the order | Monitor, review rules above 20% |
| Data completeness | Visits with both arrival and triage timestamps | 95% or higher |
| Nurse usability | System Usability Scale score | 70 or higher |

Targets are hypotheses. Final values will be set from the Phase 0 baseline.

## 11 Risks and how we will manage them

| Risk | Mitigation |
|---|---|
| Extra data entry slows nurses | Clerks do the entry. Nurses tap Start and End only, two taps per patient. |
| Power or network outage | Local server on a UPS, offline-first screens, paper tokens as fallback and automatic sync. |
| Over-reliance affects safety | Clinical triage always overrides. Red-flag fast-track. Fall back to the manual process. Weekly safety review in the pilot. |
| Patients overstate red flags | The nurse confirms at triage. Flag accuracy is audited and shared with the clerk team. |
| Privacy and consent | Minimal data, role-based access, encryption, audit log, impact assessment and a registration notice. |
| Alert fatigue and staff resistance | Co-design with nurses. Send only actionable alerts. Tune thresholds in shadow mode. |
| Patients who cannot read or have no ID | No ID needed. The door screen is spoken in English or Kiswahili. Tokens are called aloud and shown by number. |
| Poor data quality | Validation at entry, alerts for missing timestamps and a weekly data quality report. |

## 12 Where we are now and next steps

Mzima is at the concept and design stage. There is no prototype, pilot hospital or real data yet. The simulation study (build step 7) will give early evidence, clearly labelled as simulated.

1. Choose the pilot hospital and a clinical champion.
2. Confirm the triage scale in use and the current hospital information system.
3. Agree team roles and start the prototype with simulated data.
4. Check every figure cited in the application against its source.
5. Begin the ethics and data-sharing approval process.
