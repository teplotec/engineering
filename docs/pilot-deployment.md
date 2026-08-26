# TEPLOTEC Engineering Pilot Deployment

## Goal

Run a small production-like engineering pilot on the existing TEPLOTEC agency server.

The pilot is intentionally narrow. It should prove that the engineering calculation architecture is useful to engineers before we build a full engineering platform.

## Pilot architecture

```text
                         Browser
                            |
                            v
                    TEPLOTEC Engineering UI
                            |
                            v
                     Rails API / Web App
                            |
              +-------------+-------------+
              |                           |
              v                           v
        PostgreSQL                 Engineering Worker
                                          |
                              +-----------+-----------+
                              |                       |
                              v                       v
                           CoolProp              Python solvers
                              |                       |
                              +-----------+-----------+
                                          |
                                          v
                                   Calculation Run
                                          |
                                          v
                                    PostgreSQL

Optional second phase:

                    Simulation Worker
                          |
                    OpenModelica
                          |
                    OMSimulator/FMI
```

## Recommended pilot stack

### Web application

Use the team's existing Rails experience for the orchestration/API layer.

Suggested stack:

- Ruby on Rails 8+
- PostgreSQL
- React + TypeScript for the engineering UI
- Tailwind CSS
- Inertia.js or a clean JSON API boundary

The exact frontend integration is less important than keeping engineering calculations outside the UI.

### Engineering worker

Use Python for numerical calculations.

Initial dependencies:

- CoolProp
- Pydantic or equivalent input validation
- pytest
- NumPy where useful

The worker should expose a simple internal job interface or HTTP API.

### Deployment

For the pilot, Docker Compose is sufficient.

```text
compose
├── app
├── postgres
├── engineering-worker
└── nginx / reverse proxy
```

Do not introduce Kubernetes for the pilot. The engineering calculation workload is small and the first objective is correctness, traceability, and engineer usability.

## Pilot service boundaries

### `teplotec-engineering-web`

Responsibilities:

- authentication
- projects
- project revisions
- engineering forms
- calculation run history
- result presentation
- validation status
- engineer comments/approval

### `teplotec-engineering-worker`

Responsibilities:

- CoolProp calls
- thermodynamic equations
- hydraulic equations
- deterministic validation rules
- calculation result generation

The worker should not own business/project data. It receives a versioned calculation input and returns a versioned calculation result.

### PostgreSQL

Initial tables/entities:

```text
engineering_projects
engineering_project_revisions
engineering_interfaces
thermal_requirements
source_conditions
refrigeration_cycles
components
component_revisions
calculation_runs
calculation_results
validation_results
engineering_releases
```

Do not over-normalize the first schema. JSONB can be used for immutable input/output snapshots while stable searchable attributes remain first-class columns.

## Pilot API

### Create project

```http
POST /api/v1/projects
```

```json
{
  "code": "HP-GT12-PILOT-001",
  "name": "GT-12 Engineering Pilot",
  "type": "heat_pump_product"
}
```

### Save thermal requirements

```http
POST /api/v1/projects/:id/thermal-requirements
```

```json
{
  "heating_capacity_kw": 12,
  "heating_supply_c": 35,
  "heating_return_c": 30,
  "cooling_capacity_kw": null
}
```

### Define source interface

```http
POST /api/v1/projects/:id/source-interface
```

```json
{
  "fluid": "water",
  "inlet_c": 8,
  "outlet_c": 5,
  "flow_m3_h": 1.2,
  "pressure_drop_kpa": 38
}
```

### Calculate

```http
POST /api/v1/projects/:id/cycle/calculate
```

The request should contain a complete immutable input snapshot or a reference to a project revision.

The response should include:

- state points
- temperatures
- pressures
- enthalpies
- entropy
- density
- mass flow
- heating capacity
- cooling capacity
- compressor power
- COP
- superheat
- subcooling
- energy balance
- validation status

### Hydraulic calculation

```http
POST /api/v1/projects/:id/hydraulics/calculate
```

Return:

- velocity
- Reynolds number
- friction factor
- pipe pressure drop
- fitting pressure drop
- valve pressure drop
- total pressure drop
- required pump head

## First UI

The first UI should have a persistent left sidebar and a small number of pages.

```text
Project
  0 Overview
  1 Thermal Requirements
  2 Source Interface
  3 Refrigeration Cycle
  4 Hydraulics
  5 Validation
  6 Calculation History
```

Each page should contain:

1. Inputs
2. Units
3. Calculate action
4. Results
5. Validation status
6. Provenance
7. Revision/history

## Example result page

```text
GT-12 / Refrigeration Cycle

INPUTS
Heating capacity       12.0 kW
Supply temperature     35.0 C
Return temperature     30.0 C
Source inlet            8.0 C
Source outlet           5.0 C
Refrigerant             R32

RESULT
Heating capacity       12.3 kW
Compressor power        2.9 kW
COP                     4.24
Mass flow              ... kg/s

EVAPORATOR
Pressure               ... bar
Temperature            ... C
Superheat                6 K

CONDENSER
Pressure               ... bar
Temperature            ... C
Subcooling               4 K

VALIDATION
[PASS] Energy balance
[PASS] Pressure limits
[PASS] Operating envelope
[PASS] Input consistency

PROVENANCE
Engine version          0.1.0
Formula set             refrigeration-cycle-v1
Calculated              2026-08-27  ...
```

The result page is the key UX differentiator. The engineer should be able to trust a result because the system exposes how it was produced.

## Determinism requirements

The same:

- engine version
- formula set
- component revisions
- input snapshot

must produce the same calculation result within documented numerical tolerances.

Store all four as part of every calculation run.

Avoid calling external AI services from the calculation path.

## Engineering CI for the pilot

The repository should eventually include tests for known engineering points.

Example:

```text
R32 reference operating point

expected COP:              4.x
expected heating output:  12.x kW
expected energy balance:  < tolerance
expected pressure range:  within limits
```

The exact reference values must be established and reviewed by the engineering team. Do not invent acceptance values merely to make CI pass.

## Deployment stages

### Stage 1 - local

```text
Rails + PostgreSQL + Python worker
```

Run known test cases.

### Stage 2 - agency server

Deploy the same Docker Compose stack.

Expose it through a dedicated internal or protected hostname.

Use HTTPS and authentication even for the pilot if it is reachable outside a private network.

### Stage 3 - engineer pilot

Give the lead engineer a real project and compare the platform against the existing manual calculation workflow.

For every result, record:

- platform result
- engineer's existing result
- difference
- reason for difference
- whether the platform formula/model needs correction

### Stage 4 - simulation worker

Add OpenModelica only after the deterministic calculation layer has been validated.

```text
web
 |
 +--> engineering-worker
 |
 +--> simulation-worker -> OpenModelica / OMSimulator
```

This prevents simulation complexity from hiding errors in the basic engineering model.

## What success looks like

The pilot is successful if an engineer can:

1. create a heat-pump engineering project
2. enter requirements once
3. run deterministic calculations
4. inspect every important number and its provenance
5. compare revisions
6. reproduce an old calculation
7. identify validation failures
8. export/share a calculation report

Only after this is reliable should the project expand into component selection, P&ID, BOM, OpenModelica, CAD integration, ERP, and field commissioning.

## Future integration

The engineering platform should eventually connect to:

```text
Site / Drilling Engineering
          |
          v
Engineering Interface
          |
          v
Heat Pump Product Engineering
          |
          +--> Engineering BOM
          |
          v
Released Product Revision
          |
          v
ERP / Procurement / Manufacturing
          |
          v
Manufactured Unit / Serial Number
          |
          v
Commissioning / Field Data
          |
          v
Digital Twin / Service
```

The integration should be revision-aware. A site project, product engineering project, BOM, manufactured unit, and commissioning record must remain separately identifiable and linked by explicit revisions rather than becoming one large mutable record.
