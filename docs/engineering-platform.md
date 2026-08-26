# TEPLOTEC Engineering Platform

## Context

The current TEPLOTEC engineering workflow includes detailed functional/hydraulic/refrigeration schematics created by the lead engineer. The next step is to move from a drawing that answers **what is connected to what** toward an engineering model that can answer **why this topology and component sizing are correct** and **what happens when an operating parameter changes**.

The target is a semi-automated engineering platform for geothermal heat pumps and related HVAC/refrigeration equipment.

## Separation of responsibilities

### Automate on the TEPLOTEC server

#### 1. Thermodynamic properties

Use CoolProp as the property engine. The backend should be able to resolve fluid state properties from combinations of temperature, pressure, enthalpy, entropy, and quality.

Typical outputs:

- temperature
- pressure
- enthalpy
- entropy
- density
- specific heat
- viscosity
- thermal conductivity
- vapor quality

#### 2. Refrigeration-cycle calculations

Represent the basic cycle as a graph of states and components:

```text
Evaporator -> Compressor -> Condenser -> Expansion Device -> Evaporator
```

Calculate:

- evaporating pressure/temperature
- condensing pressure/temperature
- mass flow
- compressor power
- heating capacity
- cooling capacity
- COP
- superheat
- subcooling
- state-point enthalpies
- energy balance

#### 3. Hydraulics

Implement internal calculations for pipes, fittings, valves, heat exchangers, and pumps.

Calculate:

- Reynolds number
- friction factor
- pipe pressure drop
- fitting pressure drop
- valve pressure drop
- total pressure drop
- velocity
- pump head
- required flow

The goal is to make pipe and fitting sizing repeatable and testable rather than manually recalculating every project.

#### 4. Product configuration

The product model should allow an engineer to define engineering intent such as:

```text
Product: geothermal heat pump
Heating capacity: 12 kW
Refrigerant: R32
Ground inlet: 8 C
Ground outlet: 5 C
Heating supply: 35 C
Heating return: 30 C
```

The system should then derive the required operating point and candidate components.

#### 5. Component selection assistance

Maintain a normalized internal component catalog. Manufacturer tools and APIs should enrich or validate this catalog, but TEPLOTEC should not make the core engineering system dependent on one vendor API.

Component records should eventually include:

- manufacturer
- model
- component type
- supported fluids/refrigerants
- operating envelope
- performance maps
- nominal capacity
- pressure limits
- temperature limits
- connection sizes
- dimensional data
- efficiency data
- cost/availability where legally and operationally appropriate
- source and revision

Potential external sources/tools include Danfoss, Copeland, SWEP, Bitzer, Carel, Schneider and other manufacturers.

#### 6. P&ID generation

The product model should become the source for generating a first-pass P&ID/topology rather than requiring every symbol and connection to be drawn manually.

Example flow:

```text
Engineering intent
        |
        v
Product model
        |
        +--> Thermodynamic model
        +--> Hydraulic model
        +--> Component selection
        +--> P&ID topology
        +--> BOM
        +--> Engineering report
```

The engineer reviews and approves the generated result.

#### 7. BOM generation

The engineering model should generate a structured BOM from the selected components, including quantities, references, connections, and revision/traceability data.

## Keep specialized tools in the engineer workflow

### Danfoss Coolselector 2

Use for practical refrigeration component selection and independent validation, especially valves, filters, piping/component pressure drops, and manufacturer-specific component behavior.

It should be treated as a specialist validation/selection tool, not the TEPLOTEC source of truth.

### CAD

Use SolidWorks, Inventor, or an equivalent CAD system for:

- enclosure
- mounting
- physical piping arrangement
- compressor installation
- heat exchanger packaging
- electrical cabinet
- service access
- manufacturing drawings
- interference checking

CAD is not the primary thermodynamic solver.

### CFD

Do not make CFD part of every design iteration. Use it when a lower-order model indicates a real need, for example:

- poor flow distribution
- difficult heat-exchanger manifolding
- local pressure/velocity issues
- detailed air/brine/water flow behavior
- detailed heat-transfer validation

## Simulation layer

### OpenModelica

OpenModelica is the preferred open-source system simulation direction for the initial platform. It provides a programmable Modelica environment suitable for steady-state and dynamic system models.

Python can orchestrate simulations and exchange parameters/results with the simulation layer.

### OMSimulator / FMI

Use FMI and OMSimulator when multiple models need to be coupled. This allows components or subsystems to be simulated independently and connected as FMUs/co-simulation units.

Potential model decomposition:

```text
Compressor FMU
      |
      v
Refrigeration cycle FMU
      |
      +--> Heat exchanger FMU
      |
      +--> Expansion valve FMU
      |
      +--> Ground loop FMU
      |
      +--> Building load FMU
```

This is the long-term path toward a TEPLOTEC digital twin.

## Proposed technical architecture

```text
                         TEPLOTEC Engineering Platform
                                      |
                                Web Application
                                      |
                                 Rails / API
                                      |
                              PostgreSQL / Catalog
                                      |
                 +--------------------+--------------------+
                 |                    |                    |
             Calculator           Simulation           Catalog
                 |                    |                    |
              Python             OpenModelica        Components
                 |                    |
             CoolProp          OMSimulator / FMI
                 |                    |
                 +--------------------+--------------------+
                                      |
                           Reports / P&ID / BOM
```

### Suggested services

```text
teplotec-api
teplotec-engineering-worker
teplotec-simulation-worker
teplotec-openmodelica
postgres
object-storage
```

Simulation should be isolated from the main API because simulations can be expensive, long-running, and potentially require different runtime dependencies.

## API direction

The platform should expose stable engineering APIs instead of forcing UI-only workflows.

Example:

```http
POST /api/v1/thermodynamics/state
POST /api/v1/heat-pumps/simulate
POST /api/v1/hydraulics/calculate
POST /api/v1/components/select
POST /api/v1/products/:id/validate
POST /api/v1/products/:id/generate-pid
POST /api/v1/products/:id/generate-bom
```

Example heat-pump simulation input:

```json
{
  "capacity_kw": 12,
  "refrigerant": "R32",
  "ground_in_c": 8,
  "ground_out_c": 5,
  "water_in_c": 30,
  "water_out_c": 35
}
```

The API should return state points, energy balance, compressor power, capacity, COP, pressures, temperatures, mass flow, superheat, subcooling, and validation status.

## Engineering CI

Treat engineering models as versioned artifacts and test them like software.

A change to a compressor, refrigerant, heat exchanger, pipe, valve, or control parameter should be able to run automated checks.

Minimum checks:

- thermodynamic energy balance
- operating pressure limits
- compressor envelope
- COP
- heating/cooling capacity
- mass-flow consistency
- heat-exchanger capacity
- hydraulic pressure drop
- flow velocity
- safety constraints
- BOM consistency

The engineering result should be diffable between revisions.

Example:

```text
GT-12 revision A -> revision B

COP:       4.21 -> 4.38
Power:     2.85 -> 2.74 kW
Pressure:  OK
Flow:      OK
Envelope:  OK

PASS
```

## Design philosophy

The system should automate repetitive engineering calculations, not automate away engineering responsibility.

Human approval remains required for:

- final component selection
- mechanical packaging
- safety architecture
- certification-critical decisions
- CFD interpretation
- physical test validation
- release approval

The system should preserve assumptions, calculation versions, model versions, source data, and approvals for traceability.

## Strategic outcome

The long-term asset is not only the CAD library. It is a structured TEPLOTEC engineering knowledge base containing:

- validated components
- equations
- design rules
- operating envelopes
- tested configurations
- simulation models
- P&ID templates
- BOM relationships
- physical test results
- engineering decisions

This knowledge base can become the foundation for a digital twin of each TEPLOTEC heat-pump model and eventually support manufacturing, service, commissioning, and field diagnostics.
