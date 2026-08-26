# TEPLOTEC Engineering

Engineering knowledge base and simulation architecture for TEPLOTEC heat pumps, geothermal systems, refrigeration circuits, hydraulics, controls, mechanical design, BOM, P&ID, and digital twins.

## Purpose

This repository is the source of truth for engineering decisions, calculations, simulation models, component data, design rules, and research related to TEPLOTEC products.

## Engineering principle

The target is not to replace the engineer. The target is to automate the repeatable 80% of engineering work while keeping engineering review, physical design, safety decisions, validation, and final approval under human control.

> Our server calculates and orchestrates. Specialized engineering tools validate, detail, and approve.

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

This model becomes the common source for calculations, simulation, P&ID, BOM, reports, and eventually the digital twin.

## Suggested repository structure

```text
engineering/
├── docs/
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

## Current direction

Start with an open-source, programmable engineering stack rather than making a proprietary monolith. Build the calculation and product-model layer ourselves, integrate open simulation tools, and keep specialized commercial tools for validation and detailed design.
