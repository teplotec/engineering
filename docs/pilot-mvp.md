# TEPLOTEC Engineering Pilot MVP

## Objective

Build the smallest useful engineering application on the existing agency server to validate the concept with the lead engineer.

The MVP is not a complete heat-pump digital twin. It is a deterministic engineering workspace proving that structured inputs can produce reproducible thermal and hydraulic results through an API and web UI.

## MVP scope

### Included

1. Project creation
2. Product Engineering project type
3. Thermal Requirements form
4. Source/Ground interface form
5. Basic refrigeration-cycle calculator
6. Basic hydraulic calculator
7. Calculation history
8. Validation result display
9. Project/revision sidebar
10. JSON API for every calculation
11. Docker Compose deployment
12. PostgreSQL persistence
13. Automated unit tests for equations and known reference cases

### Explicitly deferred

- full OpenModelica integration
- OMSimulator/FMI orchestration
- CFD
- automatic CAD generation
- ERP write integration
- production BOM release integration
- multi-tenant authorization
- advanced manufacturer catalog synchronization

## First vertical slice

```text
Create Project
    |
    v
Thermal Requirements
    |
    v
Source / Ground
    |
    v
Refrigeration Cycle
    |
    v
Hydraulics
    |
    v
Validation
    |
    v
Calculation History
```

## Example project

Use a representative geothermal heat-pump case, initially:

```text
Product: GT-12
Heating capacity target: 12 kW
Refrigerant: R32
Ground inlet: 8 C
Ground outlet: 5 C
Heating return: 30 C
Heating supply: 35 C
```

These values are an example test case only. They must not be presented as a validated TEPLOTEC product specification.

## API

Initial endpoints:

```http
POST /api/v1/projects
GET  /api/v1/projects/:id
POST /api/v1/projects/:id/revisions
POST /api/v1/projects/:id/calculations/refrigeration-cycle
POST /api/v1/projects/:id/calculations/hydraulics
GET  /api/v1/projects/:id/calculations
POST /api/v1/projects/:id/validate
```

## Refrigeration calculator contract

Input should identify:

- refrigerant
- design heating/cooling requirement
- source-side design temperatures
- load-side design temperatures
- superheat target
- subcooling target
- compressor model or simplified compressor assumptions

Output should include:

- state points
- pressure
- temperature
- enthalpy
- mass flow
- heating capacity
- cooling capacity
- compressor power
- COP
- superheat
- subcooling
- energy balance error
- validation status

The first version may use a simplified compressor model. The model must clearly identify assumptions and must not imply manufacturer-grade compressor performance unless manufacturer performance data is loaded.

## Hydraulic calculator contract

Input:

- fluid
- concentration where applicable
- temperature
- pipe material/roughness
- diameter
- length
- fittings
- valves
- component pressure drops
- flow rate

Output:

- Reynolds number
- velocity
- friction factor
- pipe pressure drop
- component pressure drop
- total pressure drop
- required pump head
- validation warnings

## UI

Desktop-first engineering workspace with a persistent sidebar:

```text
GT-12 / Rev. 1

0 Overview                 ✓
1 Thermal Requirements     ✓
2 Source / Ground          ✓
3 Refrigeration Cycle      ●
4 Heat Exchangers          ○
5 Hydraulics               ○
6 Components               ○
7 Simulation               ○
8 P&ID                     ○
9 BOM                       ○
10 Validation              ○
11 Release                 ○
```

Each page should show:

- editable inputs
- units
- provenance/status
- calculated values
- validation messages
- calculation revision
- stale/dependency warnings

## Determinism demonstration

The pilot must demonstrate that the same immutable input snapshot and engine/rule-set version produce the same result.

The UI should expose:

```text
Calculation #42
Engine: teplotec-engineering 0.1.0
Rule set: refrigeration-cycle-v1
Inputs: Project GT-12 Rev.1
Status: PASS
```

## Deployment

Initial deployment should be intentionally simple:

```text
Docker Compose

teplotec-web/api
teplotec-engineering-worker
postgres
```

The simulation worker should be added later as a separate service because OpenModelica and FMI workloads have different runtime requirements.

## Success criteria

The pilot is successful when the lead engineer can:

1. create a project
2. enter engineering requirements
3. enter/select a source interface
4. run a deterministic calculation
5. inspect every important input and output
6. see validation failures instead of hidden bad results
7. repeat the calculation and obtain the same result
8. change one parameter and compare the engineering delta
9. export/share the calculation result

The most important success criterion is trust: the engineer must understand where every result came from and be able to reproduce it.
