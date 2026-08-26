# TEPLOTEC Engineering Workspace

## Purpose

The Engineering Workspace is the human-facing application for the TEPLOTEC engineering platform.

Its job is not to be another generic CAD or calculator application. It is the deterministic engineering interface around the TEPLOTEC engineering model, calculation engines, simulation workers, component catalog, P&ID generation, BOM generation, and engineering approvals.

The core product promise is:

> The engineer enters engineering intent once. The platform performs repeatable calculations from versioned formulas and component data, shows the assumptions and inputs, and keeps the result traceable.

The UI should make it obvious when a result is:

- calculated by a deterministic engineering rule
- imported from a manufacturer source
- produced by a simulation
- manually entered by an engineer
- awaiting engineering approval

## Do not build a generic onboarding wizard

A pure onboarding wizard is the wrong primary UX because engineering work is iterative. Engineers will return to earlier decisions, change assumptions, compare revisions, and run simulations repeatedly.

Use a **Project Workspace with a persistent sidebar** instead.

The initial project creation can be a short onboarding flow, after which the engineer works in a structured workspace.

## Workspace layout

```text
+-------------------------------------------------------------------+
| TEPLOTEC Engineering                 Project: GT-12 Kyiv Demo     |
+----------------------+--------------------------------------------+
| PROJECT              |                                            |
| 0 Overview            | Current engineering step                |
| 1 Requirements       |                                            |
| 2 Site / Ground      | Inputs / forms / calculations            |
| 3 Thermal Model      |                                            |
| 4 Refrigeration      |                                            |
| 5 Hydraulics         |                                            |
| 6 Components         |                                            |
| 7 Simulation         |                                            |
| 8 P&ID               |                                            |
| 9 BOM                |                                            |
| 10 Validation        |                                            |
| 11 Release           |                                            |
|                      |                                            |
| STATUS               | [Draft] [Calculated] [Review] [Released] |
+----------------------+--------------------------------------------+
```

The sidebar should show:

- step number
- step name
- status
- blocking errors
- completion indicator
- last calculation/revision

Do not force a linear workflow. Engineers should be able to jump between steps, while the platform prevents release when required dependencies are missing or invalid.

## Initial project creation

Project creation should capture only stable identity and scope:

- project name
- project code
- customer/site reference where applicable
- engineering owner
- project type
- target product/model where applicable
- linked projects

Do not ask for every engineering parameter during onboarding. Those belong to the appropriate engineering step.

## Project types

The platform must distinguish between different engineering scopes.

### A. Site / Installation Engineering Project

This describes the physical installation and heat source/load environment.

Typical scope:

- site
- building thermal load
- geology / ground conditions
- drilling plan
- boreholes
- ground loop
- pipe geometry
- brine/glycol
- ground-loop hydraulic design
- seasonal operating assumptions
- measured field data

Outputs relevant to a heat-pump product include:

- design ground inlet/outlet temperatures
- design brine flow
- available heat-source capacity
- design pressure drop
- fluid definition
- operating envelope
- seasonal/design-point conditions

### B. Heat Pump Product Engineering Project

This describes the heat pump machine itself.

Typical scope:

- required heating/cooling capacity
- refrigerant
- compressor
- evaporator
- condenser
- expansion device
- refrigerant circuit
- water/brine circuits
- pumps
- controls
- sensors
- safety devices
- cabinet/mechanical packaging
- P&ID
- component BOM
- simulation models

### C. Deployment / Commissioning Project

This represents a real manufactured unit installed at a site.

It should link:

```text
Product Engineering Revision
        |
        v
Manufactured Unit / Serial Number
        |
        v
Site / Installation Project
        |
        v
Commissioning + Field Measurements
```

This separation is important. A product design should be reusable across many installations, while each installation has its own ground loop, building load, commissioning data, and site conditions.

## Engineering interface between projects

Do not copy data manually between Site Engineering and Product Engineering.

Create a versioned **Engineering Interface** object.

Example:

```text
Site Project: SITE-001
Interface: SOURCE-001 rev.3

Ground fluid: 30% glycol
Design source inlet: 8.0 C
Design source outlet: 5.0 C
Design flow: 1.2 m3/h
Available source capacity: 14.5 kW
Source pressure drop: 38 kPa
Design minimum source temperature: 2.0 C

        |
        v
Heat Pump Project: HP-GT12-001
```

The heat-pump project consumes a specific released revision of the interface.

If the drilling design changes from revision 3 to revision 4, the heat-pump project should show that a dependency changed and allow the engineer to recalculate.

This creates traceability without merging two fundamentally different engineering projects.

## Recommended sidebar for the first heat-pump MVP

### 0. Overview

Show:

- project identity
- product target
- current revision
- engineering status
- linked site/source interface
- last calculation
- blocking issues

### 1. Thermal Requirements

Inputs:

- heating capacity
- cooling capacity
- design outdoor temperature if relevant
- supply temperature
- return temperature
- domestic hot water requirements if applicable
- operating modes
- target COP/SCOP where applicable

Outputs:

- design operating points
- required heat-source capacity
- thermal balance

### 2. Source / Ground Interface

For geothermal systems:

- ground-loop fluid
- inlet/outlet temperature
- flow
- pressure
- design source capacity
- linked Site Project / Interface revision

### 3. Refrigeration Cycle

Configure:

- refrigerant
- evaporating conditions
- condensing conditions
- superheat
- subcooling
- compressor
- expansion device
- cycle topology

Show a state-point table:

```text
Point | T | P | h | s | density | quality
```

### 4. Heat Exchangers

Configure and calculate:

- evaporator
- condenser
- UA
- approach temperatures
- flow rates
- pressure drops
- heat-transfer duty

### 5. Hydraulics

Calculate:

- pipe sizes
- lengths
- fittings
- valves
- heat-exchanger pressure drops
- total pressure drop
- pump head
- flow velocity

### 6. Components

Show selected and candidate components.

Every component should have:

- source
- revision
- operating envelope
- selection reason
- calculation inputs
- validation status

### 7. Simulation

Provide:

- steady-state simulation
- operating-point sweep
- transient simulation later
- comparison between revisions

### 8. P&ID

Generate a first-pass P&ID from the engineering model.

The engineer can annotate/revise it, but topology should originate from structured engineering data whenever possible.

### 9. BOM

Generate engineering BOM from the selected configuration.

Do not directly overwrite ERP data.

Use a controlled release flow:

```text
Engineering BOM
      |
      v
Engineering Review
      |
      v
Released BOM revision
      |
      v
ERP / Procurement / Manufacturing BOM
```

### 10. Validation

Run deterministic checks:

- thermodynamic energy balance
- pressure limits
- compressor envelope
- flow limits
- heat-exchanger capacity
- hydraulic pressure drop
- safety constraints
- missing component data
- BOM consistency
- unresolved assumptions

### 11. Release

Release an immutable engineering revision.

A release should capture:

- engineering model revision
- component revisions
- formula/calculation engine version
- simulation version
- input snapshot
- generated P&ID
- BOM
- reports
- approvals

## Deterministic engineering result model

Every calculation result must be traceable to its inputs and engine version.

Example:

```json
{
  "calculation": "heat_pump_cycle",
  "engine": "teplotec-engineering",
  "engine_version": "0.1.0",
  "formula_set": "refrigeration-cycle-v1",
  "inputs_revision": "HP-GT12-001:7",
  "component_revisions": [
    "COMPRESSOR-X:3",
    "HX-EVAP-X:2",
    "HX-COND-X:4"
  ],
  "status": "passed"
}
```

This is the mechanism that makes the platform more trustworthy than an ad-hoc Google search, spreadsheet, calculator, or AI answer.

## API-first design

The UI should be a client of the engineering APIs.

Initial API surface:

```text
POST /api/v1/projects
GET  /api/v1/projects/:id

POST /api/v1/projects/:id/thermal-requirements
POST /api/v1/projects/:id/source-interface
POST /api/v1/projects/:id/cycle/calculate
POST /api/v1/projects/:id/hydraulics/calculate
POST /api/v1/projects/:id/components/select
POST /api/v1/projects/:id/simulations
POST /api/v1/projects/:id/pid/generate
POST /api/v1/projects/:id/bom/generate
POST /api/v1/projects/:id/validate
POST /api/v1/projects/:id/release
```

The same APIs should be usable by:

- web UI
- future CLI
- automated engineering CI
- ERP integration
- service/commissioning tools
- AI assistants as a controlled interface

AI must not be the calculation authority. AI can help prepare inputs, explain results, or suggest alternatives, but final numerical results must come from deterministic engineering engines and validated component data.

## UX principle: explain every number

When an engineer sees:

```text
COP = 4.18
```

the UI should make it possible to inspect:

- formula/model used
- input values
- refrigerant properties source
- component data revisions
- calculation engine version
- assumptions
- validation checks
- previous revision result

The platform should never present a naked number without provenance.

## MVP principle

The first version should not attempt to model every heat pump component.

Build one deterministic vertical slice:

```text
Create project
    -> enter thermal requirements
    -> define source conditions
    -> select refrigerant
    -> calculate basic refrigeration cycle
    -> calculate basic hydraulics
    -> show state points + COP + pressure drops
    -> validate
    -> save immutable calculation run
```

Once this is trusted, add component selection, P&ID, BOM, OpenModelica simulation, and integration with site engineering.
