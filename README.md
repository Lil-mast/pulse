# Mzima Queue & Flow Optimisation

Mzima is an agentic patient-flow co-pilot for hospital outpatient triage. It helps shift leads and outpatient department managers anticipate operational bottlenecks and make better use of the staff and triage capacity already available.

## The problem

Public hospital outpatient queues are often managed largely by arrival order, even when arrivals surge and the number of available triage staff changes during the day. Patients can accumulate faster than nurses can process them while capacity elsewhere goes unused.

This operational gap matters in African health systems, where health workers are constrained and unevenly distributed. WHO estimates that East Africa had only 34% of the health workforce it needed in 2024. Kenyan hospital research has also linked staff shortages and stretched capacity with delayed or ineffective triage.

## What Mzima does

Mzima monitors operational signals such as:

- Patient arrival rates and queue length
- Waiting time and time to first assessment
- Triage-station status and service times
- Staff availability
- Historical arrival patterns

An AI planning agent uses these signals to forecast near-term demand, identify approaching bottlenecks, evaluate operational options, and recommend an action to the shift lead. Example recommendations include:

- Opening an available triage station before an expected surge
- Reallocating authorised staff between stations
- Balancing arrivals across available assessment points
- Staggering staff breaks
- Escalating when time to first assessment is rising

The goal is to use existing staff and capacity more intelligently so fewer patients spend unnecessary time waiting for their first assessment.

## Human control and clinical boundaries

Mzima is an operational support tool. It does **not** diagnose, prescribe, score clinical urgency, or decide who gets seen. It can recommend and notify, but a nurse or clinician remains responsible for clinical triage and for deciding whether to act on a recommendation.

## Intended users

- Triage nurses
- Shift leads
- Outpatient department managers

Patients benefit from a shorter and more predictable wait to first assessment.

## Proposed technology stack

The repository does not yet contain application code or framework configuration, so the following is a proposed starting stack rather than a description of implemented technology:

| Area | Suggested technology | Purpose |
| --- | --- | --- |
| Web application | Next.js, React, TypeScript | Responsive operations dashboard for shift leads and triage staff |
| UI styling | Tailwind CSS | Consistent, accessible dashboard components |
| API | Python, FastAPI | Operational data ingestion and recommendation endpoints |
| Forecasting and planning | Python, pandas, scikit-learn; rules-based constraints around recommendations | Near-term demand forecasts and transparent operational planning |
| Data storage | PostgreSQL | Queue, station, staffing, and recommendation records |
| Background work | Redis and Celery | Scheduled forecasts, event processing, and notifications |
| Deployment | Docker | Reproducible development and deployment environments |

Clinical safety, privacy, explainability, and reliable operation should guide technology choices. Recommendations should expose their operational basis, preserve a human approval step, and avoid using patient-identifying data unless it is necessary and appropriately protected.

## Project status

This repository currently contains project documentation only. The stack above is a suggested foundation; no application implementation is present yet.

## Getting started

There is no application to install or run yet. Once the implementation is added, this section should document prerequisites, setup, configuration, and development commands.

## License

See [LICENSE](LICENSE).
