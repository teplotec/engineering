# TEPLOTEC Engineering Roadmap

## Phase 0 - Pilot foundation

- repository structure
- engineering domain model
- deterministic calculation contracts
- Docker Compose
- PostgreSQL
- API skeleton
- engineering workspace UI

## Phase 1 - Deterministic calculators

- CoolProp integration
- refrigerant state calculations
- basic refrigeration cycle
- hydraulic pipe/fitting calculations
- validation rules
- calculation snapshots and revisioning

## Phase 2 - Component intelligence

- normalized component catalog
- compressor performance maps
- heat exchanger data
- expansion valve data
- pumps and valves
- candidate selection
- manufacturer data provenance

## Phase 3 - Product engineering

- reusable product families
- GT-08 / GT-12 / GT-16 configurations
- generated P&ID topology
- engineering BOM
- engineering reports
- engineering CI

## Phase 4 - Dynamic simulation

- OpenModelica worker
- Modelica system models
- FMI/OMSimulator coupling
- steady-state comparison against deterministic calculators
- transient operating scenarios

## Phase 5 - Site integration

- Site Engineering project
- drilling/borehole data interface
- ground-loop design
- measured source conditions
- versioned Site Interface consumed by Product Engineering

## Phase 6 - Manufacturing and field lifecycle

```text
Engineering Release
      |
      v
ERP/MES
      |
      v
Manufacturing
      |
      v
Serialised Unit
      |
      v
Commissioning
      |
      v
Field Telemetry
      |
      v
Digital Twin
```

Integrations should be downstream of an approved engineering release. ERP must not become the source of engineering design intent.

## First implementation order

Do not start with CFD, advanced CAD automation, or full ERP integration.

Build in this order:

1. project/revision model
2. engineering workspace UI
3. CoolProp service
4. refrigeration-cycle calculator
5. hydraulic calculator
6. validation engine
7. calculation history/provenance
8. component catalog
9. P&ID/BOM generation
10. OpenModelica integration

## Guiding rule

Every phase should produce a usable vertical slice and preserve deterministic, versioned engineering data. Avoid building a large framework before the lead engineer has used the first calculator on a real design case.
