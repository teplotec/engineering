# TEPLOTEC Engineering Data Model

## Principle

The engineering model is the source of truth shared by calculations, simulations, P&ID, BOM, reports, manufacturing integration, and commissioning.

The model must separate:

- requirement
- assumption
- measured fact
- calculated result
- selected component
- approved/released decision

## Core entities

```text
EngineeringProject
  ├── ProjectRevision
  ├── Requirement
  ├── Assumption
  ├── SiteInterface
  ├── ProductConfiguration
  ├── Topology
  ├── ComponentSelection
  ├── CalculationRun
  ├── SimulationRun
  ├── ValidationRun
  ├── EngineeringBOM
  └── Release
```

## Project separation

### Site Engineering Project

Represents the installation/source side.

Examples:

- site
- boreholes
- drilling results
- ground loop
- geology
- measured temperatures
- flow and pressure data

### Product Engineering Project

Represents a reusable heat pump product design.

Examples:

- GT-08
- GT-12
- GT-16

A product revision can consume a generic source envelope or a concrete Site Engineering interface.

### Deployment Project

Represents a physical installation of a released product.

It references both:

- released product revision
- released/site-specific source interface

It then stores actual commissioning measurements.

## Versioned interfaces

Site and Product projects should communicate through explicit versioned interfaces.

Example:

```text
Site Interface SI-001 rev.3

source_type: vertical_borehole
fluid: glycol_30
inlet_temperature_c: 8
outlet_temperature_c: 5
flow_m3_h: 1.2
available_head_kpa: 80
```

A product configuration records exactly which interface revision it consumed.

If the interface changes, dependent calculations become stale and must be rerun.

## CalculationRun

Every deterministic calculation should record:

- project revision
- calculation type
- engine name/version
- formula/rule-set version
- input snapshot
- component revisions
- output snapshot
- validation status
- timestamp

The input snapshot must be immutable after the run.

## Provenance

Every engineering value should eventually have provenance metadata:

```text
source_type:
  requirement | assumption | measured | calculated | catalog

source_reference:
  document, sensor, catalog record, calculation run, etc.

source_revision:
  revision identifier

confidence/status:
  draft | verified | approved | released
```

## Staleness

A calculation becomes stale when a dependency changes.

Examples:

- source temperature changed
- refrigerant changed
- compressor revision changed
- heat exchanger changed
- pipe size changed
- equation/rule-set version changed

The UI should surface stale results explicitly rather than silently recalculating or silently presenting old results.

## BOM relationship

Engineering BOM is generated from the released product configuration.

```text
Product Configuration rev.7
        |
        v
Engineering BOM rev.7
        |
        v
ERP/MES released BOM
```

Engineering owns design intent and configuration. ERP owns downstream procurement/manufacturing execution after release.
