# Traffic Light Constraint Language (TLCL) Specification

## Table of Contents

1. [Introduction](#1-introduction)
2. [Overview](#2-overview)
3. [Multi-Level Configuration Architecture](#3-multi-level-configuration-architecture)
4. [Program Structure](#4-program-structure)
5. [Signal Definitions](#5-signal-definitions)
6. [Constraint Syntax](#6-constraint-syntax)
7. [Temporal Operators](#7-temporal-operators)
8. [Helper Functions](#8-helper-functions)
9. [Constraint Types](#9-constraint-types)
10. [Configuration Levels](#10-configuration-levels)
11. [Constraint Merging](#11-constraint-merging)
12. [Semantics](#12-semantics)
13. [Verification](#13-verification)
14. [Complete Examples](#14-complete-examples)
15. [Appendices](#15-appendices)

---

## 1. Introduction

### 1.1 Purpose

The Traffic Light Constraint Language (TLCL) is a formal, declarative language for specifying the behavior and safety requirements of traffic light controllers. TLCL allows traffic engineers to express timing constraints, safety requirements, and optimization goals in a clear, verifiable format.

### 1.2 Design Goals

- **Expressiveness**: Capture real-world traffic control requirements
- **Verifiability**: Enable automated proof of safety properties
- **Simplicity**: More readable than pure temporal logic
- **Domain-specific**: Optimized for traffic light scenarios
- **Implementation-independent**: Separates specification from execution
- **Extensibility**: Room for future enhancements
- **Multi-level**: Support regional, intersection, and program-level configuration

### 1.3 Key Principles

1. **Separation of Concerns**: Specifications describe *what* should happen, not *how* to achieve it
2. **Hard and Soft Constraints**: Distinguish between mandatory safety requirements and optimization goals
3. **Hierarchical Configuration**: Regional regulations, intersection topology, and program timing are managed separately
4. **Vendor Independence**: Any compliant TLC runtime can execute TLCL programs
5. **Formal Verification**: Programs can be proven correct before deployment
6. **Constraint-Based**: No fixed cycles or state sequences - runtime finds valid executions

### 1.4 Relationship to Existing Work

TLCL builds on:
- **Metric Temporal Logic (MTL)** for timing constraints
- **Timed Automata** theory for state machine semantics
- **RSMP Nordic** traffic light controller specifications
- Existing fixed-time program formats

---

## 2. Overview

### 2.1 What is TLCL?

TLCL is a constraint-based language for traffic light control. Instead of specifying exact sequences of states and transitions, engineers specify:

- What signals exist
- What constraints must always hold (safety)
- What goals should be optimized (efficiency, responsiveness)

The TLC runtime is responsible for finding execution strategies that satisfy all hard constraints while optimizing soft constraints.

**Important**: TLCL does not prescribe fixed cycles, phases, or state sequences. The runtime dynamically determines behavior based on constraints and current conditions.

### 2.2 Core Concepts

#### Signals
Outputs (lights) and inputs (sensors, buttons, detectors)

#### States
Named configurations of signal values (optional convenience)

#### Hard Constraints
Safety-critical requirements that MUST always be satisfied. Violation indicates a program error.

#### Soft Constraints
Optimization goals that SHOULD be satisfied. Violations are acceptable but the runtime tries to minimize them. Each soft constraint has a priority (1-10).

#### Temporal Logic
Constraints use temporal operators to express requirements about timing and sequencing over time.

---

## 3. Multi-Level Configuration Architecture

TLCL supports three configuration levels, each managed by different stakeholders with different change frequencies:

### 3.1 Three Levels

```
┌─────────────────────────────────────────────────┐
│ REGIONAL LEVEL                                  │
│ (National/State Regulations)                    │
│                                                 │
│ - Managed by: Traffic authorities               │
│ - Change frequency: Rarely (years)              │
│ - Scope: All controllers in region              │
│ - Contains: Safety standards, regulations       │
└─────────────────────────────────────────────────┘
                    ↓ inherits
┌─────────────────────────────────────────────────┐
│ INTERSECTION LEVEL                              │
│ (Site Configuration)                            │
│                                                 │
│ - Managed by: Site engineers                    │
│ - Change frequency: Occasionally (months)       │
│ - Scope: One physical intersection              │
│ - Contains: Topology, signals, hardware mapping │
└─────────────────────────────────────────────────┘
                    ↓ inherits
┌─────────────────────────────────────────────────┐
│ PROGRAM LEVEL                                   │
│ (Individual Timing Programs)                    │
│                                                 │
│ - Managed by: Traffic operators                 │
│ - Change frequency: Frequently (daily/weekly)   │
│ - Scope: One timing pattern                     │
│ - Contains: Timing, optimization goals          │
└─────────────────────────────────────────────────┘
```

### 3.2 Benefits

| Benefit | Description |
|---------|-------------|
| **Separation of Concerns** | Different stakeholders manage different aspects |
| **Reuse** | One regional config serves thousands of intersections |
| **Consistency** | National standards automatically enforced everywhere |
| **Flexibility** | Easy to create new programs without modifying infrastructure |
| **Safety** | Regional regulations cannot be overridden |
| **Maintainability** | Changes happen at the appropriate level |

---

## 4. Program Structure

### 4.1 File Format

TLCL programs are written in YAML or JSON format with `.tlcl` extension.

### 4.2 Top-Level Schema

All three levels share a similar structure, differing in which sections they include:

```yaml
# Regional Level
regional:
  id: string
  name: string
  version: string
  authority: string
  jurisdiction: string
  signal_types: [...]
  hard_constraints: [...]
  soft_constraints: [...]
  parameters: {...}

# Intersection Level
intersection:
  id: string
  name: string
  version: string
  location: {...}
  inherits:
    - regional: string
  signals: [...]
  conflicts: [...]
  topology: {...}
  hard_constraints: [...]
  soft_constraints: [...]
  parameters: {...}

# Program Level
program:
  id: string
  name: string
  version: string
  description: string
  inherits:
    - intersection: string
  schedule: {...}
  hard_constraints: [...]
  soft_constraints: [...]
  initial_state: {...}
  parameters: {...}
```

### 4.3 Metadata Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier |
| `name` | string | Yes | Human-readable name |
| `version` | string | Yes | Semantic version (e.g., "1.0.0") |
| `description` | string | No | Detailed description |

---

## 5. Signal Definitions

Signals are defined at the **intersection level** and represent physical inputs and outputs.

### 5.1 Signal Schema

```yaml
signals:
  - id: string              # Unique identifier (e.g., "ns_red")
    type: enum              # "light" | "input" | "detector"
    description: string     # Human-readable description
    group: string           # Optional grouping (e.g., "north_south")
    hardware_address: string # Physical I/O address
    initial: boolean        # Initial value (default: false)
```

### 5.2 Signal Types

| Type | Description | Examples |
|------|-------------|----------|
| `light` | Output signal (lamp) | Red, yellow, green lights |
| `input` | Digital input | Pedestrian buttons, manual switches |
| `detector` | Vehicle/presence detector | Loop detectors, radar sensors |

### 5.3 Example

```yaml
signals:
  - id: ns_red
    type: light
    description: "North-South red light"
    group: north_south
    hardware_address: "output_1"
    initial: true
    
  - id: ped_button_ns
    type: input
    description: "North-South pedestrian button"
    hardware_address: "input_5"
    
  - id: detector_ns_approach
    type: detector
    description: "North approach vehicle detector"
    hardware_address: "input_1"
```

---

## 6. Constraint Syntax

### 6.1 Constraint Structure

```yaml
constraints:
  - id: string              # Unique constraint identifier
    name: string            # Human-readable name
    description: string     # Detailed description
    type: "hard" | "soft"   # Constraint type (if ambiguous from section)
    priority: number        # For soft constraints (1-10, higher = more important)
    formula: string         # TLCL formula
    tags: [string]          # Optional categorization
    regulation_ref: string  # Reference to regulation (for regional constraints)
```

### 6.2 Formula Language

TLCL formulas use a restricted subset of temporal logic with traffic-specific operators.

#### 6.2.1 Basic Boolean Expressions

```
signal_id                    # Signal is on/true
not(expr)                    # Logical negation
expr and expr                # Logical AND
expr or expr                 # Logical OR
expr implies expr            # Logical implication (→)
```

**Examples:**
```yaml
formula: "ns_red"
formula: "not(ns_green)"
formula: "ns_red and not(ns_green)"
formula: "ped_button implies eventually(ped_walk)"
```

#### 6.2.2 Comparison Operators

```
duration(signal) < value
duration(signal) <= value
duration(signal) == value
duration(signal) >= value
duration(signal) > value
```

---

## 7. Temporal Operators

Temporal operators allow expressing requirements about timing and sequencing.

### 7.1 Operator Reference

| Operator | Syntax | Description | MTL Equivalent |
|----------|--------|-------------|----------------|
| **Always** | `always(φ)` | φ must be true at all times | □φ |
| **Eventually** | `eventually(φ)` | φ must become true eventually | ◊φ |
| **Eventually Within** | `eventually_within(a, b, φ)` | φ must occur within [a,b] seconds | ◊[a,b]φ |
| **When-Then** | `when(φ) then ψ` | If φ occurs, then ψ must follow | φ → ψ |
| **Stays** | `stays(φ, t)` | φ remains true for exactly t seconds | - |
| **Stays At Least** | `stays_at_least(φ, t)` | φ remains true for ≥ t seconds | - |
| **Stays At Most** | `stays_at_most(φ, t)` | φ remains true for ≤ t seconds | - |
| **Until** | `φ until ψ` | φ holds until ψ becomes true | φ U ψ |

### 7.2 Usage Examples

#### Always
```yaml
# Red and green never both on
formula: "always(not(ns_red and ns_green))"

# At least one light is always on
formula: "always(ns_red or ns_yellow or ns_green)"
```

#### Eventually Within
```yaml
# Yellow light lasts 3-5 seconds
formula: "always(when(rising_edge(yellow)) then eventually_within(3, 5, falling_edge(yellow)))"

# Respond to pedestrian button within 90 seconds
formula: "always(when(rising_edge(ped_button)) then eventually_within(0, 90, ped_walk))"
```

#### Stays At Least / At Most
```yaml
# Green must stay on for at least 30 seconds
formula: "always(when(rising_edge(green)) then stays_at_least(green, 30))"

# Yellow must not exceed 5 seconds
formula: "always(when(rising_edge(yellow)) then stays_at_most(yellow, 5))"
```

---

## 8. Helper Functions

Helper functions simplify common patterns in traffic light constraints.

### 8.1 Function Reference

| Function | Signature | Description |
|----------|-----------|-------------|
| `exactly_one` | `[bool] → bool` | Exactly one argument is true |
| `at_most_one` | `[bool] → bool` | At most one argument is true |
| `count_true` | `[bool] → int` | Count how many arguments are true |
| `duration` | `signal → float` | How long (seconds) signal has been true |
| `time_since` | `signal → float` | Seconds since signal was last true |
| `rising_edge` | `signal → bool` | Signal just transitioned false→true |
| `falling_edge` | `signal → bool` | Signal just transitioned true→false |
| `changed` | `signal → bool` | Signal value changed this tick |

### 8.2 Usage Examples

#### Signal Mutual Exclusion
```yaml
# Exactly one light per direction
formula: "always(exactly_one([ns_red, ns_yellow, ns_green]))"
```

#### Edge Detection
```yaml
# Respond to button press (rising edge)
formula: "always(when(rising_edge(ped_button)) then eventually_within(0, 90, ped_walk))"

# Yellow must follow green (not red)
formula: "always(when(rising_edge(yellow)) then falling_edge(green))"
```

#### Duration Tracking
```yaml
# Green has been on for at least 30 seconds
formula: "duration(green) >= 30"

# Maximum red time is 120 seconds
formula: "always(duration(red) <= 120)"
```

---

## 9. Constraint Types

### 9.1 Hard Constraints

Hard constraints are **safety-critical** and MUST always be satisfied. Any violation is considered a program error and the runtime MUST halt or raise an alarm.

#### Common Hard Constraints

| Category | Examples |
|----------|----------|
| **Signal Exclusion** | Only one light on per direction |
| **Cross-Traffic Safety** | Conflicting directions never both green |
| **Timing Bounds** | Yellow 3-5 seconds, minimum red time |
| **Sequencing** | Yellow only follows green, not red |
| **All-Red Periods** | Mandatory clearance between conflicts |

#### Example Hard Constraints

```yaml
hard_constraints:
  - id: safety_mutual_exclusion
    name: "Signal Mutual Exclusion"
    description: "Only one light can be on per direction"
    formula: "always(exactly_one([ns_red, ns_yellow, ns_green]))"
    tags: [safety, exclusion]
    
  - id: safety_cross_traffic
    name: "Cross Traffic Safety"
    description: "Conflicting movements cannot both be green"
    formula: "always(not(ns_green and ew_green))"
    tags: [safety, conflict]
    
  - id: timing_yellow_min
    name: "Yellow Minimum Duration"
    description: "Yellow light must last at least 3 seconds"
    formula: "always(when(rising_edge(yellow)) then stays_at_least(yellow, 3))"
    tags: [safety, timing, yellow]
    
  - id: timing_yellow_max
    name: "Yellow Maximum Duration"
    description: "Yellow light must not exceed 5 seconds"
    formula: "always(when(rising_edge(yellow)) then stays_at_most(yellow, 5))"
    tags: [safety, timing, yellow]
```

### 9.2 Soft Constraints

Soft constraints are **optimization goals**. The runtime SHOULD try to satisfy them, but violations are acceptable. Each soft constraint has a **priority** (1-10).

#### Priority Levels

| Priority | Level | Description | Use Cases |
|----------|-------|-------------|-----------|
| **10** | Critical | Must satisfy unless impossible | Emergency vehicle preemption, critical safety margins |
| **7-9** | High | Very important operational goals | Pedestrian response time, main traffic flow |
| **4-6** | Medium | Important but flexible | Green time targets, balanced service |
| **1-3** | Low | Nice-to-have optimizations | Minimize idle time, energy efficiency |

#### Example Soft Constraints

```yaml
soft_constraints:
  - id: green_duration_target
    name: "Green Duration Target"
    description: "Green should last 30-60 seconds"
    priority: 8
    formula: "always(when(rising_edge(green)) then eventually_within(30, 60, falling_edge(green)))"
    tags: [timing, optimization]
    
  - id: pedestrian_response
    name: "Pedestrian Response Time"
    description: "Respond to pedestrian button within 90 seconds"
    priority: 10
    formula: "always(when(rising_edge(ped_button)) then eventually_within(0, 90, ped_walk))"
    tags: [pedestrian, responsiveness]
    
  - id: max_wait_time
    name: "Maximum Wait Time"
    description: "Each direction should get green at least every 120 seconds"
    priority: 7
    formula: "always(when(falling_edge(ns_green)) then eventually_within(0, 120, rising_edge(ns_green)))"
    tags: [fairness, responsiveness]
    
  - id: balanced_green
    name: "Balanced Green Time"
    description: "Try to give similar green times to both directions"
    priority: 5
    formula: "abs(total_green_ns - total_green_ew) <= 15"
    tags: [fairness, balance]
```

---

## 10. Configuration Levels

### 10.1 Regional Level

**Purpose:** Define national or regional traffic control standards and regulations.

**Managed by:** Traffic authorities, government agencies

**Change frequency:** Rarely (years)

**Scope:** All controllers in the jurisdiction

#### Regional Level Structure

```yaml
regional:
  id: string
  name: string
  version: string
  authority: string          # Governing body
  jurisdiction: string       # Geographic scope
  effective_date: string     # When standard takes effect
  
  signal_types: [...]        # Standard signal type definitions
  hard_constraints: [...]    # Mandatory safety requirements
  soft_constraints: [...]    # Recommended practices
  parameters: {...}          # Standard parameters
```

#### What Goes in Regional Config

- National safety standards (yellow duration, all-red minimums)
- Regulatory requirements
- Standard signal types
- Pedestrian crossing standards
- Accessibility requirements
- Legal compliance constraints

#### Example

```yaml
regional:
  id: "se_national_v2023"
  name: "Swedish National Traffic Light Standards 2023"
  version: "2023.1"
  authority: "Trafikverket"
  jurisdiction: "Sweden"
  effective_date: "2023-01-01"
  
  hard_constraints:
    - id: reg_yellow_duration
      name: "National Yellow Duration Standard"
      description: "Yellow light must be 3-5 seconds per national regulation"
      formula: "always(when(rising_edge(yellow)) then stays_at_least(yellow, 3) and stays_at_most(yellow, 5))"
      regulation_ref: "TKV-2023-45.2"
```

### 10.2 Intersection Level

**Purpose:** Define physical topology, signal groups, and site-specific constraints.

**Managed by:** Site engineers, intersection designers

**Change frequency:** Occasionally (when hardware changes)

**Scope:** One physical intersection

#### Intersection Level Structure

```yaml
intersection:
  id: string
  name: string
  version: string
  location: {...}            # Geographic location
  
  inherits:
    - regional: string       # Reference to regional config
  
  signals: [...]             # Physical signals at this site
  conflicts: [...]           # Movement conflict matrix
  topology: {...}            # Physical measurements
  hard_constraints: [...]    # Site-specific safety requirements
  soft_constraints: [...]    # Site-specific optimization goals
  parameters: {...}          # Site-specific parameters
```

#### What Goes in Intersection Config

- Signal definitions (all physical inputs/outputs)
- Hardware addresses/mappings
- Conflict matrix (which movements conflict)
- Physical topology (crossing distances, sight lines)
- Site-specific safety constraints
- Signal group definitions

#### Example

```yaml
intersection:
  id: "stockholm_main_elm_001"
  name: "Main Street & Elm Avenue"
  version: "1.2.0"
  location:
    city: "Stockholm"
    coordinates: [59.3293, 18.0686]
    
  inherits:
    - regional: "se_national_v2023"
  
  signals:
    - id: ns_red
      type: light
      hardware_address: "output_1"
      group: north_south
      
  conflicts:
    - movements: [ns_green, ew_green]
      type: direct_conflict
      
  topology:
    crossing_width_ns: 12  # meters
    approach_speed_ns: 50  # km/h
```

### 10.3 Program Level

**Purpose:** Define specific timing patterns and optimization strategies.

**Managed by:** Traffic operators

**Change frequency:** Frequently (daily, weekly, seasonal)

**Scope:** One timing program for one intersection

#### Program Level Structure

```yaml
program:
  id: string
  name: string
  version: string
  description: string
  
  inherits:
    - intersection: string   # Reference to intersection config
  
  schedule: {...}            # When this program is active
  soft_constraints: [...]    # Program-specific timing goals
  initial_state: {...}       # Starting state
  parameters: {...}          # Program-specific parameters
```

#### What Goes in Program Config

- Timing optimization goals
- Green time targets
- Service level preferences
- Time-of-day schedules
- Special event configurations
- Adaptive strategies

**Note:** Programs typically do NOT add hard constraints. They inherit all safety requirements from regional and intersection levels.

#### Example

```yaml
program:
  id: "stockholm_main_elm_001_morning_rush"
  name: "Morning Rush Hour"
  version: "1.5.2"
  
  inherits:
    - intersection: "stockholm_main_elm_001"
  
  schedule:
    active_days: [monday, tuesday, wednesday, thursday, friday]
    active_hours: "07:00-09:30"
    
  soft_constraints:
    - id: prog_ns_green_priority
      name: "NS Green Priority"
      description: "Give more green time to NS during morning rush"
      priority: 9
      formula: "always(when(rising_edge(ns_green)) then eventually_within(45, 75, falling_edge(ns_green)))"
```

---

## 11. Constraint Merging

### 11.1 Merging Process

When a program is loaded, constraints from all levels are merged:

```
Final Configuration = Regional ⊕ Intersection ⊕ Program
```

### 11.2 Merging Rules

#### 1. Hard Constraints
All hard constraints from all levels are combined (logical AND).

```
All Hard = Regional.Hard ∪ Intersection.Hard ∪ Program.Hard
```

A program must satisfy **all** hard constraints from **all** levels.

#### 2. Soft Constraints
All soft constraints are combined with their priorities preserved.

```
All Soft = Regional.Soft ∪ Intersection.Soft ∪ Program.Soft
```

The runtime optimizes the weighted sum of all soft constraint violations.

#### 3. Signals
Signals are defined **only** at the intersection level.

- Regional level: defines signal **types**
- Intersection level: defines actual **signals**
- Program level: **cannot** define new signals, only reference existing ones

#### 4. Parameters
Lower levels can override parameters from higher levels.

```
Final Parameters = Regional.Params ← Intersection.Params ← Program.Params
```

Where `←` means "overridden by".

### 11.3 Inheritance Example

```yaml
# Regional defines safety standard
regional:
  hard_constraints:
    - id: reg_yellow_duration
      formula: "always(when(rising_edge(yellow)) then stays_at_least(yellow, 3))"

# Intersection defines physical constraints
intersection:
  inherits: [regional: "se_national_v2023"]
  signals: [ns_red, ns_yellow, ns_green, ew_red, ew_yellow, ew_green]
  hard_constraints:
    - id: int_ns_ew_conflict
      formula: "always(not(ns_green and ew_green))"

# Program defines timing goals
program:
  inherits: [intersection: "stockholm_main_elm_001"]
  soft_constraints:
    - id: prog_ns_green_target
      priority: 9
      formula: "always(when(rising_edge(ns_green)) then eventually_within(45, 75, falling_edge(ns_green)))"

# Merged result contains:
# - 1 hard constraint from regional
# - 1 hard constraint from intersection
# - 1 soft constraint from program
# = All must be satisfied
```

### 11.4 Conflict Detection

Conflicts are detected across all levels during validation:

- **Syntax conflicts:** Two constraints cannot both be satisfied
- **Timing conflicts:** Impossible timing requirements
- **Signal conflicts:** Reference to undefined signals
- **Parameter conflicts:** Incompatible parameter values

**Example conflict:**
```yaml
# Regional says:
hard_constraints:
  - formula: "always(stays_at_least(green, 30))"

# Program says (CONFLICT):
hard_constraints:
  - formula: "always(stays_at_most(green, 20))"

# This is unsatisfiable (cannot be ≥30 and ≤20 simultaneously)
```

---

## 12. Semantics

### 12.1 Time Model

- Time advances in discrete **ticks**
- Default tick rate: **100 milliseconds**
- All timing constraints use **seconds** as the unit
- Clocks maintain precise timing for temporal operators

### 12.2 Signal Model

- Signals are **boolean** (on/off, true/false)
- Signals can only change at **tick boundaries**
- All signal changes in a tick are **atomic** (happen simultaneously)
- Constraints are evaluated **after** signal updates

### 12.3 Constraint Evaluation

#### Hard Constraints
- Evaluated at **every tick**
- **Any violation** immediately fails the program
- Runtime **MUST** halt or raise alarm on hard constraint violation
- No recovery from hard constraint violation

#### Soft Constraints
- Evaluated at **every tick**
- Violations are **logged** with priority and duration
- Runtime **attempts** to minimize violations
- Cost function: `Σ(priority × violation_duration)`
- Soft constraint violations are **acceptable**

### 12.4 Execution Model

At each tick, the runtime:

1. **Update clocks** - Advance all timing counters
2. **Process inputs** - Read sensors, buttons, detectors
3. **Evaluate constraints** - Check all hard and soft constraints
4. **Decide action** - Determine next state (optimization problem)
5. **Update outputs** - Set signal values
6. **Log state** - Record for monitoring and verification

### 12.5 No Fixed Cycles

**Important**: TLCL does not prescribe fixed cycles or phase sequences. The runtime dynamically determines which signals to activate and when, based solely on satisfying constraints.

- There is no predetermined "cycle" (e.g., NS green → yellow → red → EW green...)
- The runtime may adapt behavior based on inputs, detectors, or optimization
- Timing patterns emerge from constraint satisfaction, not fixed programming

---

## 13. Verification

### 13.1 Purpose

Programs should be verified **before deployment** to ensure:
- Hard constraints are satisfiable
- No conflicting requirements
- All signal references are valid
- Timing requirements are achievable

### 13.2 Verification Levels

#### Level 1: Static Analysis (Fast)
- Syntax checking
- Type checking
- Basic conflict detection
- Signal reference validation

#### Level 2: Bounded Model Checking (Moderate)
- Simulate execution for N steps (typically 100-1000)
- Try different input scenarios
- Check for constraint violations
- Generate counterexamples if conflicts found

#### Level 3: Formal Verification (Slow but Complete)
- Use SMT solvers (e.g., Z3)
- Prove properties mathematically
- Provide formal correctness guarantees
- Integration with tools like UPPAAL

### 13.3 Verification Workflow

```
1. Parse TLCL file → Check syntax
2. Merge configurations → Regional + Intersection + Program
3. Validate signals → All references exist
4. Check hard constraints → No conflicts
5. Analyze soft constraints → Warn if unsatisfiable
6. Generate report → Pass/Fail with details
```

### 13.4 Conflict Examples

**Contradictory constraints:**
```yaml
# These conflict (cannot both be satisfied):
- formula: "always(ns_red)"
- formula: "always(ns_green)"
```

**Impossible timing:**
```yaml
# Cannot satisfy both:
- formula: "always(stays(ns_green, 60))"      # Must be exactly 60s
- formula: "always(stays_at_most(ns_green, 30))"  # Max 30s
```

**Undefined signal:**
```yaml
# Error if 'ns_blue' not defined in intersection config:
- formula: "always(ns_blue)"
```

---

## 14. Complete Examples

### 14.1 Regional Configuration Example

```yaml
regional:
  id: "se_national_v2023"
  name: "Swedish National Traffic Light Standards 2023"
  version: "2023.1"
  authority: "Trafikverket"
  jurisdiction: "Sweden"
  effective_date: "2023-01-01"
  
  signal_types:
    - id: standard_vehicle_light
      outputs: [red, yellow, green]
      
    - id: pedestrian_light
      outputs: [red, green]
  
  hard_constraints:
    - id: reg_yellow_duration
      name: "National Yellow Duration Standard"
      description: "Yellow light must be 3-5 seconds per national regulation"
      formula: "always(when(rising_edge(yellow)) then stays_at_least(yellow, 3) and stays_at_most(yellow, 5))"
      regulation_ref: "TKV-2023-45.2"
      tags: [national_standard, timing, yellow]
      
    - id: reg_all_red_minimum
      name: "Mandatory All-Red Period"
      description: "Minimum 1 second all-red between conflicting movements"
      formula: "always(when(conflict_transition) then stays_at_least(all_red, 1))"
      regulation_ref: "TKV-2023-47.1"
      tags: [national_standard, safety, all_red]
      
    - id: reg_minimum_green
      name: "Minimum Green Duration"
      description: "Any green phase must be at least 5 seconds"
      formula: "always(when(rising_edge(green)) then stays_at_least(green, 5))"
      regulation_ref: "TKV-2023-45.8"
      tags: [national_standard, timing, green]
  
  soft_constraints:
    - id: reg_rec_max_wait
      name: "Recommended Maximum Wait Time"
      description: "Each direction should get service at least every 120 seconds"
      priority: 7
      formula: "always(when(falling_edge(green)) then eventually_within(0, 120, rising_edge(green)))"
      regulation_ref: "TKV-2023-REC-12"
      tags: [national_recommendation, service_level]
  
  parameters:
    pedestrian_walking_speed: 1.2  # m/s
```

### 14.2 Intersection Configuration Example

```yaml
intersection:
  id: "stockholm_main_elm_001"
  name: "Main Street & Elm Avenue"
  version: "1.2.0"
  location:
    city: "Stockholm"
    coordinates: [59.3293, 18.0686]
    
  inherits:
    - regional: "se_national_v2023"
  
  signals:
    # North-South vehicle signals
    - id: ns_red
      type: light
      hardware_address: "output_1"
      group: north_south
      initial: true
      
    - id: ns_yellow
      type: light
      hardware_address: "output_2"
      group: north_south
      
    - id: ns_green
      type: light
      hardware_address: "output_3"
      group: north_south
      
    # East-West vehicle signals
    - id: ew_red
      type: light
      hardware_address: "output_4"
      group: east_west
      initial: true
      
    - id: ew_yellow
      type: light
      hardware_address: "output_5"
      group: east_west
      
    - id: ew_green
      type: light
      hardware_address: "output_6"
      group: east_west
      
    # Pedestrian signals
    - id: ped_ns_red
      type: light
      hardware_address: "output_7"
      group: pedestrian_ns
      initial: true
      
    - id: ped_ns_green
      type: light
      hardware_address: "output_8"
      group: pedestrian_ns
      
    # Inputs
    - id: ped_button_ns
      type: input
      hardware_address: "input_5"
      
    - id: detector_ns_approach
      type: detector
      hardware_address: "input_1"
  
  conflicts:
    - movements: [ns_green, ew_green]
      type: direct_conflict
      
    - movements: [ns_green, ped_ns_green]
      type: partial_conflict
      
  topology:
    crossing_width_ns: 12  # meters
    crossing_width_ew: 15  # meters
    sight_distance_ns: 85  # meters
    approach_speed_ns: 50  # km/h
  
  hard_constraints:
    - id: int_signal_exclusion_ns
      name: "NS Signal Mutual Exclusion"
      description: "Only one NS vehicle light at a time"
      formula: "always(exactly_one([ns_red, ns_yellow, ns_green]))"
      tags: [intersection_specific, hardware]
      
    - id: int_signal_exclusion_ew
      name: "EW Signal Mutual Exclusion"
      description: "Only one EW vehicle light at a time"
      formula: "always(exactly_one([ew_red, ew_yellow, ew_green]))"
      tags: [intersection_specific, hardware]
      
    - id: int_ns_ew_conflict
      name: "NS-EW Conflict Safety"
      description: "NS and EW cannot both be green"
      formula: "always(not(ns_green and ew_green))"
      tags: [intersection_specific, topology]
      
    - id: int_ped_clearance_ns
      name: "NS Pedestrian Clearance Time"
      description: "Calculated from actual crossing width (12m)"
      formula: "pedestrian_clearance_time >= 10"
      tags: [intersection_specific, pedestrian]
  
  soft_constraints:
    - id: int_balance_directions
      name: "Balance NS and EW Green Time"
      description: "Try to give similar total green time to both directions over time"
      priority: 5
      formula: "abs(total_green_ns - total_green_ew) <= 15"
      tags: [intersection_specific, fairness]
  
  parameters:
    controller_type: "Swarco ITC-3"
    firmware_version: "2.4.1"
    tick_rate: 100  # ms
```

### 14.3 Program Configuration Example

```yaml
program:
  id: "stockholm_main_elm_001_morning_rush"
  name: "Main & Elm - Morning Rush Hour"
  version: "1.5.2"
  description: "Optimized for heavy NS traffic during morning commute"
  
  inherits:
    - intersection: "stockholm_main_elm_001"
  
  schedule:
    active_days: [monday, tuesday, wednesday, thursday, friday]
    active_hours: "07:00-09:30"
    
  soft_constraints:
    - id: prog_ns_green_priority
      name: "NS Green Priority (Morning Rush)"
      description: "Give more green time to NS during morning rush"
      priority: 9
      formula: "always(when(rising_edge(ns_green)) then eventually_within(45, 75, falling_edge(ns_green)))"
      tags: [program_specific, timing, rush_hour]
      
    - id: prog_ew_green_shorter
      name: "EW Green Reduced (Morning Rush)"
      description: "Shorter EW green during morning rush"
      priority: 8
      formula: "always(when(rising_edge(ew_green)) then eventually_within(20, 35, falling_edge(ew_green)))"
      tags: [program_specific, timing, rush_hour]
      
    - id: prog_ns_max_wait
      name: "NS Maximum Wait Time"
      description: "NS direction should get green at least every 100 seconds"
      priority: 7
      formula: "always(when(falling_edge(ns_green)) then eventually_within(0, 100, rising_edge(ns_green)))"
      tags: [program_specific, service_level]
      
    - id: prog_ped_response
      name: "Pedestrian Response Time"
      description: "Try to respond to pedestrian buttons within 60 seconds"
      priority: 6
      formula: "always(when(rising_edge(ped_button_ns)) then eventually_within(0, 60, ped_ns_green))"
      tags: [program_specific, pedestrian]
  
  initial_state:
    ns_red: true
    ew_red: true
```

---

## 15. Appendices

### Appendix A: Temporal Logic Primer

For readers unfamiliar with temporal logic:

**Temporal logic** extends classical boolean logic with operators that describe properties over time:

- **□φ** (always φ): φ is true at all time points
- **◊φ** (eventually φ): φ becomes true at some future time point
- **φ U ψ** (φ until ψ): φ remains true until ψ becomes true
- **φ → ψ** (φ implies ψ): if φ is true, then ψ must also be true

**Metric Temporal Logic (MTL)** adds timing constraints:

- **◊[a,b]φ**: φ becomes true within the time interval [a,b] seconds

TLCL provides a more readable syntax for these operators, optimized for traffic control.

### Appendix B: Grammar (Simplified EBNF)

```
program        ::= level ":" body
level          ::= "regional" | "intersection" | "program"
body           ::= metadata signals? constraints? parameters?

constraint     ::= "id:" ID "formula:" formula
formula        ::= temporal_expr | boolean_expr | comparison

temporal_expr  ::= "always(" formula ")"
                 | "eventually(" formula ")"
                 | "eventually_within(" NUMBER "," NUMBER "," formula ")"
                 | "when(" formula ") then" formula
                 | "stays_at_least(" signal "," NUMBER ")"
                 | "stays_at_most(" signal "," NUMBER ")"

boolean_expr   ::= signal
                 | "not(" formula ")"
                 | formula "and" formula
                 | formula "or" formula
                 | "exactly_one([" signal_list "])"

comparison     ::= "duration(" signal ")" op NUMBER
op             ::= "<" | "<=" | "==" | ">=" | ">"
```

### Appendix C: Comparison with Other Approaches

| Approach | Pros | Cons |
|----------|------|------|
| **Fixed-time programs** | Simple, predictable | Inflexible, cannot express constraints |
| **State machines** | Direct control | Complex to verify, hard to modify |
| **Imperative code** | Maximum flexibility | Difficult to verify, vendor lock-in |
| **TLCL (this spec)** | Verifiable, flexible, vendor-independent | Requires sophisticated runtime |

### Appendix D: Constraint-Based vs. Fixed Cycles

**Traditional Fixed-Cycle Approach:**
- Program specifies exact sequence: State1 (30s) → State2 (5s) → State3 (40s) → ...
- Predictable but inflexible
- Cannot adapt to conditions
- Easy to understand and implement

**TLCL Constraint-Based Approach:**
- Program specifies what must be true (constraints)
- Runtime finds valid execution satisfying all constraints
- Can adapt to inputs, detectors, priorities
- More complex to implement but more powerful

**Example comparison:**

*Fixed-cycle:*
```yaml
states:
  0: "NS_GREEN"   # 30 seconds
  30: "NS_YELLOW" # 5 seconds
  35: "EW_GREEN"  # 40 seconds
  75: "EW_YELLOW" # 5 seconds
  # Repeat
```

*Constraint-based (TLCL):*
```yaml
soft_constraints:
  - id: ns_green_target
    formula: "always(when(rising_edge(ns_green)) then eventually_within(30, 60, falling_edge(ns_green)))"
  - id: ew_green_target
    formula: "always(when(rising_edge(ew_green)) then eventually_within(30, 60, falling_edge(ew_green)))"
  - id: responsive_service
    formula: "always(when(falling_edge(green)) then eventually_within(0, 90, rising_edge(green)))"
```

The constraint-based approach gives the runtime freedom to find the best solution.

### Appendix E: Glossary

| Term | Definition |
|------|------------|
| **Constraint** | A requirement that must (hard) or should (soft) be satisfied |
| **Signal** | A physical input or output (light, button, detector) |
| **Temporal Logic** | Logic extended with operators for reasoning about time |
| **MTL** | Metric Temporal Logic - temporal logic with timing constraints |
| **Hard Constraint** | Safety-critical requirement that MUST be satisfied |
| **Soft Constraint** | Optimization goal that SHOULD be satisfied |
| **Tick** | One time step in the discrete time model (typically 100ms) |
| **Runtime** | Software that executes TLCL programs |
| **Verification** | Process of proving a program is correct |
| **Conflict** | Two constraints that cannot both be satisfied |
| **Constraint-Based** | Approach where runtime finds behavior satisfying constraints, rather than following fixed sequence |

### Appendix F: References

- **Metric Temporal Logic**: Koymans, R. (1990). "Specifying real-time properties with metric temporal logic"
- **Timed Automata**: Alur, R. & Dill, D. (1994). "A theory of timed automata"
- **Temporal Logic Planning**: Rahmani & O'Kane (2020). "What to Do When You Can't Do It All: Temporal Logic Planning with Soft Constraints"
- **RSMP Protocol**: RSMP Nordic (2023). "Roadside Message Protocol Specification"
- **Traffic Control Standards**: Various national and international traffic control standards
