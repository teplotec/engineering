# TEPLOTEC Engineering

Engineering knowledge base and simulation architecture for TEPLOTEC heat pumps, geothermal systems, refrigeration circuits, hydraulics, controls, mechanical design, BOM, P&ID, and digital twins.

## Purpose

This repository is the source of truth for engineering decisions, calculations, simulation models, component data, design rules, and research related to TEPLOTEC products.

The engineering platform is intended to turn repeatable engineering knowledge into deterministic, versioned software while keeping engineering responsibility with the engineer.

## Engineering principle

The target is not to replace the engineer. The target is to automate the repeatable 80% of engineering work while keeping engineering review, physical design, safety decisions, validation, and final approval under human control.

> Our server calculates and orchestrates. Specialized engineering tools validate, detail, and approve.

## Product vision

The engineer should enter engineering intent once and receive reproducible calculations from versioned formulas, component data, and simulation models.

The platform must clearly distinguish:

- deterministic calculated values
- manufacturer-sourced data
- simulation results
- manually entered values
- engineering assumptions
- approved/released values

AI may help explain results or prepare inputs, but AI is never the numerical calculation authority.

## Engineering project model

TEPLOTEC engineering must distinguish between different engineering scopes.

### Site / Installation Engineering

Covers building load, geology, drilling, boreholes, ground loops, fluids, hydraulic design, seasonal source conditions, and field measurements.

### Heat Pump Product Engineering

Covers the heat pump machine: thermal requirements, refrigeration circuit, compressor, heat exchangers, expansion device, hydraulics, controls, safety, mechanical packaging, P&ID, and product BOM.

### Deployment / Commissioning

Links a released product revision and manufactured unit to a real installation, commissioning measurements, and service data.

These projects are connected through explicit, versioned engineering interfaces. A site design should not be copied into a product design as untraceable fields, and a product design should be reusable across many installations.

## Initial architecture

- **Python + CoolProp** - thermophysical properties and fast engineering calculations.
- **OpenModelica** - system-level thermodynamic and dynamic simulation.
- **OMSimulator / FMI** - coupling independent simulation models and FMUs.
- **PostgreSQL** - engineering data, component catalog, model versions, calculation runs, BOM, and traceability.
- **Git** - version control for engineering models, assumptions, equations, and design decisions.
- **Object storage** - CAD files, simulation artifacts, reports, and other large binary assets.
- **Rails/API layer** - engineering orchestration, product/configuration API, workflow, and UI backend.
- **Danfoss Coolselector 2 and manufacturer tools** - component selection and independent engineering validation rather than the core calculation engine.
- **CAD tools such as SolidWorks/Inventor** - mechanical design and manufacturing drawings.
- **CFD tools** - used selectively when flow distribution or detailed heat-transfer behavior requires CFD validation.

## Core automation targets

| Engineering area | Target automation | Primary approach |
|---|---:|---|
| Thermodynamics | 90-100% | Python + CoolProp |
| Refrigeration cycle | 80-90% | OpenModelica + Python |
| Hydraulics | 80-90% | Internal solver + component data |
| Component selection | 50-80% | Internal catalog + manufacturer data/tools |
| P&ID generation | 70-90% | Product model -> generated topology/P&ID |
| BOM generation | 90-100% | Product model + component catalog |
| Mechanical CAD | 20-40% | CAD tools + engineer |
| CFD | 10-30% | Specialized CFD tools when justified |

## Deterministic calculation principle

Every calculation result must be reproducible from:

- engine version
- formula/model version
- input snapshot/revision
- component data revisions

The platform should preserve provenance for every important number and expose it to the engineer.

## Initial engineering model

A heat-pump product should be represented as a structured model rather than only as a drawing. The model should capture:

- product configuration
- refrigeration topology
- heat exchangers
- compressor
- expansion device
- pumps
- valves and fittings
- pipes and dimensions
- sensors and safety devices
- fluids and refrigerants
- operating points
- temperature, pressure, enthalpy, mass flow, heat flow, and power at defined nodes
- component operating envelopes
- control parameters
- BOM references
- P&ID tags
- calculation and simulation versions
- engineering assumptions and source/provenance
- approvals and release status

This model becomes the common source for calculations, simulation, P&ID, BOM, reports, and eventually the digital twin.

## Engineering Workspace

The primary UX is a persistent project workspace, not a linear onboarding wizard.

Suggested sidebar:

```text
0 Overview
1 Thermal Requirements
2 Source / Ground Interface
3 Refrigeration Cycle
4 Heat Exchangers
5 Hydraulics
6 Components
7 Simulation
8 P&ID
9 BOM
10 Validation
11 Release
```

Project creation is intentionally small. Engineering data is entered in the relevant step and can be revised later.

See [Engineering Workspace](docs/engineering-workspace.md) for the detailed UX and project model.

## Suggested repository structure

```text
engineering/
├── docs/
│   ├── engineering-platform.md
│   ├── engineering-workspace.md
│   └── pilot-deployment.md
├── models/
│   ├── gt-08/
│   ├── gt-12/
│   └── gt-16/
├── components/
├── thermodynamics/
├── hydraulics/
├── simulations/
├── pid/
├── bom/
└── tests/
```

## Engineering CI concept

Engineering changes should eventually be testable like software changes. A compressor, heat exchanger, refrigerant, pipe size, or control parameter change should trigger automated checks for:

- thermodynamic balance
- pressure limits
- compressor operating envelope
- COP
- flow rates
- heat-exchanger capacity
- hydraulic pressure drop
- safety constraints
- BOM consistency

The output should provide a comparable engineering delta, for example:

```text
COP:       4.21 -> 4.38
Power:     2.85 -> 2.74 kW
Pressure:  OK
Flow:      OK
Envelope:  OK

PASS
```

## Pilot deployment

The first deployment should be deliberately small and run on the existing agency server using Docker Compose:

```text
web/API + PostgreSQL + engineering-worker
```

OpenModelica/OMSimulator should be added as a separate simulation worker after the deterministic calculation layer is validated.

The first vertical slice is:

```text
Create project
 -> Thermal Requirements
 -> Source Interface
 -> Refrigeration Cycle
 -> Hydraulics
 -> Validation
 -> Calculation History
```

See [Pilot Deployment](docs/pilot-deployment.md).

## ERP and manufacturing boundary

Engineering BOM should not directly mutate ERP BOM data.

The intended release flow is:

```text
Engineering BOM
      -> Engineering Review
      -> Released BOM Revision
      -> ERP / Procurement / Manufacturing
```

This keeps engineering revisions, procurement data, manufacturing structures, and field units traceable.

## Current direction

Start with an open-source, programmable engineering stack rather than making a proprietary monolith. Build the calculation and product-model layer ourselves, integrate open simulation tools, and keep specialized commercial tools for validation and detailed design.

The long-term asset is the TEPLOTEC engineering knowledge base: validated components, equations, design rules, operating envelopes, tested configurations, simulation models, P&ID templates, BOM relationships, physical test results, and engineering decisions.
