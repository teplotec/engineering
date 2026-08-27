# TEPLOTEC Engineering Workflow

## Goal

Provide an engineer-facing workflow that turns engineering intent into a versioned, deterministic, reviewable engineering result.

The UI is a workspace, not a one-time wizard. Engineers can revisit any stage, change inputs, inspect calculations, and approve revisions.

## Project types

TEPLOTEC should distinguish three related but different engineering contexts:

1. **Site / Installation Engineering**
   - site conditions
   - geology and drilling data
   - boreholes
   - ground loop topology
   - source temperatures
   - flow and hydraulic constraints

2. **Heat Pump Product Engineering**
   - thermal requirements
   - refrigeration cycle
   - compressor
   - heat exchangers
   - expansion device
   - hydraulics
   - controls
   - mechanical packaging
   - component selection

3. **Deployment / Commissioning**
   - a concrete manufactured unit
   - serial number
   - installed configuration
   - as-built parameters
   - commissioning measurements
   - field validation

These contexts are linked through versioned engineering interfaces rather than duplicated data.

## Workspace navigation

```text
Project
│
├── 0. Overview
├── 1. Thermal Requirements
├── 2. Source / Ground
├── 3. Refrigeration Cycle
├── 4. Heat Exchangers
├── 5. Hydraulics
├── 6. Components
├── 7. Simulation
├── 8. P&ID
├── 9. BOM
├── 10. Validation
└── 11. Release
```

Each stage has a status:

- Not started
- In progress
- Needs review
- Blocked
- Complete
- Superseded

The sidebar should show validation errors and unresolved dependencies without hiding the engineer's ability to navigate directly to another stage.

## Stage definitions

### 0. Overview

Create or select the project and product family.

Inputs:

- project name
- project type
- product/model
- revision
- engineer
- engineering objective

Outputs:

- project identity
- current revision
- dependency summary

### 1. Thermal Requirements

Capture the engineering intent.

Typical inputs:

- heating capacity
- cooling capacity
- supply temperature
- return temperature
- design outdoor/building conditions where relevant
- DHW requirement
- operating modes
- design points

The system should distinguish user-provided requirements from calculated values.

### 2. Source / Ground

For geothermal products, consume a versioned Site Engineering interface where available.

Typical values:

- source type
- ground/brine fluid
- inlet temperature
- outlet temperature
- design flow
- available pressure/head
- borehole count
- borehole depth
- loop topology
- design assumptions

If Site Engineering is incomplete, the product model may use explicit assumptions, but those assumptions must be marked as assumptions and must not silently become measured facts.

### 3. Refrigeration Cycle

Define the refrigeration topology and operating points.

Typical components:

- compressor
- condenser
- expansion device
- evaporator
- receiver/accumulator where required
- filter/drier
- sensors and safety devices

Calculate:

- pressures
- temperatures
- enthalpies
- mass flow
- heating capacity
- cooling capacity
- compressor power
- COP
- superheat
- subcooling
- energy balance

### 4. Heat Exchangers

Size and validate evaporator and condenser requirements.

Track:

- duty
- inlet/outlet temperatures
- flow rates
- UA or equivalent performance parameters
- pressure drop
- selected component
- manufacturer data source

### 5. Hydraulics

Calculate source and load-side hydraulic behavior.

Track:

- pipe dimensions
- lengths
- fittings
- valves
- heat exchanger pressure drops
- flow rates
- velocities
- total pressure drop
- pump head
- pump operating point

### 6. Components

Select concrete components from the normalized TEPLOTEC catalog.

Each selection should record:

- component ID
- manufacturer
- model
- catalog revision
- data source
- selection criteria
- selected operating point
- engineer approval

### 7. Simulation

Run steady-state and, later, dynamic simulations.

The first implementation should support deterministic calculation runs. OpenModelica/OMSimulator should be introduced as a separate simulation worker once the product model and calculation contracts are stable.

### 8. P&ID

Generate a first-pass topology and P&ID from the engineering model.

The engineer reviews and approves the generated representation.

### 9. BOM

Generate an engineering BOM from the released component configuration.

Do not make ERP the source of engineering truth. Engineering owns the design BOM; ERP/MES consumes an approved/released BOM.

### 10. Validation

Run all deterministic checks.

Minimum checks:

- thermodynamic balance
- operating pressure limits
- compressor envelope
- heating/cooling capacity
- mass-flow consistency
- heat exchanger duty
- hydraulic pressure drop
- flow velocity
- safety constraints
- component compatibility
- BOM consistency
- unresolved assumptions/dependencies

### 11. Release

Create a versioned engineering release.

A release should freeze:

- inputs
- assumptions
- equations/rule-set version
- component revisions
- calculation engine version
- simulation model version
- outputs
- validation results
- approvals

## Deterministic engineering principle

A calculation result must be reproducible from its recorded inputs and versions.

AI may help explain, search, or propose alternatives, but AI output must not be treated as the authoritative engineering calculation.

Authoritative results come from versioned equations, property libraries, component data, and deterministic solvers.
