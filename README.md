# the-one-scenario-corpus

Working documents: [explicación (ES)](README.es.md) · [checklist de fases (ES)](CHECKLIST.es.md)

## Project objective

**the-one-scenario-corpus** is a research project to design, generate, analyse, and select a **corpus of reproducible scenarios for The ONE (Opportunistic Network Environment)** so that DTN/OppNet routing protocols can be compared **under the same experimental conditions**.

The central idea is to keep three things clearly separate:

1. the **scenario conditions**, which describe the environment in which communication takes place;
2. the **routing protocol**, which is the experimental variable changed when algorithms are compared;
3. the **simulation results**, which are used only to evaluate protocol behaviour and not to decide which scenarios belong to the corpus.

A corpus scenario must therefore be runnable repeatedly with different routers without changing the remaining conditions:

```text
Scenario_i + EpidemicRouter
Scenario_i + ProphetRouter
Scenario_i + SprayAndWaitRouter
Scenario_i + ...
```

The final aim is to be able to state that observed differences between protocols come from protocol behaviour under the same conditions, rather than from having used different scenarios.

---

# Current methodological status

> **METHODOLOGICAL STATUS**

This project starts from a scientific methodology defined from scratch in order to build a **reproducible and representative scenario corpus for The ONE Simulator**.

The methodology is organised around eight fundamental questions:

1. what a scenario is, exactly;
2. which parameters stay constant across the whole corpus;
3. which parameters may vary between scenarios;
4. how each scenario is represented by a feature vector;
5. how scenarios of different duration, scale, or origin are normalised;
6. how similarity, redundancy, diversity, and coverage are measured between scenarios;
7. how a representative final corpus is selected from a pool of candidates;
8. how different routing protocols are later compared under exactly the same experimental conditions.

Methodological development will be incremental: the scenario space and its features are defined first, the mathematical methods of comparison and selection are studied next, and the corpus is built and validated last.

Current status:

```text
METHODOLOGY = V1 IN DESIGN
FINAL CORPUS = NOT BUILT
FINAL NUMBER OF SCENARIOS = TBD
FEATURE SPACE = TBD
NORMALIZATION CONTRACT = TBD
SIMILARITY / REDUNDANCY METRICS = TBD
SELECTION / CLUSTERING METHOD = TBD
ROUTING BENCHMARK = NOT YET AUTHORIZED
```

---

# 1. Research question

The overall question of the project is:

> **How can a finite scenario corpus for The ONE be built so that it covers the space of relevant DTN/OppNet conditions in a sufficiently diverse and non-redundant way, and so that different routing protocols can be compared under exactly the same experimental conditions?**

The problem has two distinct parts.

## 1.1 Corpus construction

We start from a potentially very large space of `.settings` configurations.

That space must be turned into a finite set that:

- covers different conditions;
- avoids duplicates and scenarios that are practically equivalent;
- preserves relevant extreme cases;
- is computationally executable;
- is traceable;
- is not built to favour any particular protocol.

## 1.2 Routing benchmark

Once the corpus is frozen:

```text
Scenario corpus
        ↓
Scenario 001 ── Router A / Router B / Router C / ...
Scenario 002 ── Router A / Router B / Router C / ...
...
Scenario N   ── Router A / Router B / Router C / ...
```

Protocols must run on the same scenarios and with the same initial conditions whenever the semantics of the simulator allow it.

---

# 2. What we mean by a scenario

In this methodology, the **routing protocol is not part of scenario identity**.

A scenario is defined conceptually as:

\[
S_i = (E, M_i, C_i)
\]

where:

- \(E\): global experimental constants;
- \(M_i\): spatial and movement profile;
- \(C_i\): communication and traffic profile.

The routing protocol is applied afterwards:

\[
Run_{i,r} = (S_i, R_r)
\]

The same \(S_i\) can therefore be evaluated with multiple \(R_r\).

---

# 3. Functional decomposition of a `.settings` file

For this project, The ONE parameters are divided conceptually into five blocks.

```text
SCENARIO
│
├── A. Experimental constants
│
├── B. Spatial + Movement profile
│
├── C. Communication + Traffic profile
│
├── D. Routing protocol
│
└── E. Observation / Reports
```

## 3.1 A. Experimental constants

These are parameters we want to fix globally so that scenarios can be compared.

Initial candidates:

```text
MovementModel.worldSize = 5000,5000
Scenario.endTime = 86400        # 24 h
Scenario.updateInterval = TBD
MovementModel.warmup = TBD
seed policy = TBD
```

**Note:** `MovementModel.worldSize` is expressed in the simulator's spatial units. In the movement models used by The ONE, the unit is metres.

The exact value of each constant will be frozen only when it is justified.

## 3.2 B. Spatial + Movement profile

This describes where the nodes are and how they move.

It may include:

- one or more WKT maps;
- number of nodes;
- node groups;
- type of each group;
- initial position;
- movement model;
- speed;
- wait times;
- routes;
- points of interest;
- schedules;
- external traces;
- parameters specific to models such as `WorkingDayMovement`.

Examples of relevant models:

- `RandomWaypoint`;
- `RandomWalk`;
- `ShortestPathMapBasedMovement`;
- `MapRouteMovement`;
- `BusMovement`;
- `WorkingDayMovement`;
- `ExternalMovement`;
- `ExternalPathMovement`;
- others available in, or implemented by, the project.

## 3.3 C. Communication + Traffic profile

This describes communication opportunities and the generated load.

### Communication

It may include:

- number of interfaces;
- interface type;
- transmission range;
- transmission speed;
- scan interval;
- buffer size;
- TTL.

### Message traffic

It may include:

- who sends;
- who may receive;
- when messages are generated;
- interval between messages;
- temporal distribution;
- message size;
- total amount/load;
- multiple generators;
- inter-group traffic.

## 3.4 D. Routing protocol

This is the **main experimental variable of the later benchmark**.

Examples:

- `EpidemicRouter`;
- `ProphetRouter`;
- `SprayAndWaitRouter`;
- `MaxPropRouter`;
- `FirstContactRouter`;
- `DirectDeliveryRouter`;
- custom protocols.

The choice of router **must not be used to decide which scenarios belong to the corpus**.

## 3.5 E. Observation / Reports

Reports are used to evaluate simulation outcomes.

Examples:

- delivery probability;
- latency;
- overhead;
- hop count;
- contact times;
- inter-contact times;
- buffer occupancy;
- message copies;
- density;
- other KPIs frozen for the benchmark.

These results belong to the routing benchmark and not to the initial corpus-selection process, unless a future methodology explicitly justifies it.

---

# 4. Vector representation of a scenario

Each scenario must be transformed into a feature vector:

\[
\mathbf{x}_i = [x_{i1}, x_{i2}, \dots, x_{ip}]
\]

The vector must represent the relevant scenario conditions **without including the identity or the results of the routing protocol**.

The exact feature definition is still open.

## 4.1 Possible feature families

### Space / map

- dimensions;
- spatial density;
- road-graph properties;
- coverage;
- network length;
- topology;
- other properties that are scientifically relevant.

### Movement

- total number of nodes;
- number of groups;
- proportion of node types;
- speeds;
- wait times;
- movement model;
- mobility parameters;
- temporal activity;
- trace-derived metrics when appropriate.

### Communication

- range;
- transmission speed;
- number/type of interfaces;
- buffer;
- TTL.

### Traffic

- message creation rate;
- size;
- temporal distribution;
- source/destination distribution;
- load density;
- heterogeneity across groups.

## 4.2 Numeric and categorical features

Not every variable in a `.settings` file is numeric.

Example:

```text
movementModel = RandomWaypoint
movementModel = WorkingDayMovement
movementModel = ExternalMovement
```

The feature space must therefore define how to represent:

- continuous variables;
- discrete variables;
- min/max ranges;
- categorical variables;
- multiple groups;
- hierarchical structures.

No final encoding will be applied until its mathematical consequences have been reviewed.

---

# 5. Independence, similarity, and redundancy

Methodology v2 must avoid incorrectly using expressions such as:

> "a corpus of linearly independent scenarios"

without a precise mathematical definition.

If scenarios are represented by vectors of dimension \(p\), at most \(p\) linearly independent vectors can exist in \(\mathbb{R}^p\).

Therefore, for a corpus with \(N > p\), **linear independence of every scenario cannot be required**.

The correct methodological question is closer to:

> Does the corpus cover the feature space sufficiently without containing unnecessary redundancy?

## 5.1 Methods that must be studied before one is selected

The first mathematical phase will analyse, at least:

- Pearson;
- Spearman;
- covariance;
- variance;
- the correlation matrix;
- Euclidean / Manhattan / cosine distances, and other justified distances;
- SVD and rank;
- PCA;
- numerical condition;
- nearest-neighbour distance;
- clustering;
- silhouette;
- coverage measures;
- saturation criteria.

Each method must be documented with:

1. which question it answers;
2. which assumptions it requires;
3. which feature types it admits;
4. which preprocessing it needs;
5. when it can produce misleading conclusions;
6. whether it will be used for features, scenarios, clusters, or selection.

**A metric will not be fixed merely because code already exists to compute it.**

---

# 6. Feature-space normalisation

Features can lie on very different scales:

```text
nodes = 100
speed = 1.2 m/s
worldSize = 5000 m
messageSize = 1000000 bytes
range = 20 m
endTime = 86400 s
```

Comparing them directly with a distance would produce results dominated by the variables with the largest numeric scale.

The methodology must study and freeze a normalisation procedure before clustering or selection.

Possible methods to evaluate:

- min-max;
- z-score;
- robust scaling;
- logarithmic transforms when they are justified;
- a specific encoding for categorical variables;
- normalisation by relevant physical quantities.

Normalisation is part of the scientific method and must be versioned.

---

# 7. Normalisation of scenarios and real traces

Besides normalising vectors, the project needs to study the **physical/temporal normalisation of scenarios**.

Example:

```text
Standard corpus duration = 24 h
Available real trace      = 12 h
```

A trace cannot simply be duplicated or scaled without defining which property is to be preserved.

If time is transformed by:

\[
t' = s_t t
\]

and space by:

\[
\mathbf{p}' = s_x \mathbf{p}
\]

effective speed transforms as:

\[
v' = v \frac{s_x}{s_t}
\]

A **Normalization Contract** is therefore required, stating which invariants we want to preserve for each kind of transformation.

Possible invariants:

- speed;
- duration;
- spatial density;
- relative distances;
- message generation rate;
- total message volume;
- hourly patterns;
- contacts;
- others.

Status:

```text
NORMALIZATION_CONTRACT = TBD
```

Real datasets will not be normalised definitively until this contract is closed.

---

# 8. Generation of scenario pools

The new design will not generate the final corpus directly.

**Pools of candidate scenarios** will be created first.

```text
Space definition
        ↓
Candidate generation
        ↓
Pool of .settings
        ↓
Feature extraction
        ↓
Normalisation
        ↓
Mathematical analysis
        ↓
Clustering / coverage / redundancy
        ↓
Representative selection
        ↓
Final corpus
```

A pool may contain many more configurations than the final corpus.

Generation must record:

- scenario origin;
- effective parameters;
- seed;
- generation rules;
- generator version;
- normalisation applied;
- a hash or reproducible identifier when appropriate.

---

# 9. Selection of the final corpus

The size of the final corpus **is not fixed yet**.

```text
TARGET_CORPUS_SIZE = TBD
```

Values such as 50, 70, or 100 may be studied as experimental budgets, but they must not become a scientific criterion by themselves.

The final selection must justify, at the same time:

- coverage of the space;
- separation between representatives;
- redundancy;
- preservation of extremes;
- representation of clusters;
- stability of the selection;
- later computational cost.

One possible future architecture is:

```text
Pool
 ↓
Feature matrix
 ↓
Normalize
 ↓
Distance / similarity matrix
 ↓
Clustering
 ↓
Representative selection
 ↓
Coverage / saturation check
 ↓
Final corpus
```

The concrete algorithms remain `TBD` until the mathematical review is complete.

---

# 10. Equality of conditions in the benchmark

Once a scenario \(S_i\) is frozen, every router must use the same base scenario.

Conceptually:

```text
S_i
├── Epidemic
├── Prophet
├── SprayAndWait
├── MaxProp
└── ...
```

The configuration must be designed so that changing the router does not accidentally change:

- the map;
- mobility;
- nodes;
- traffic;
- interfaces;
- the relevant seeds;
- simulation time;
- the required reports.

Parameters that belong to each router will be documented separately.

---

# 11. Recommended repository structure

The current structure may be migrated gradually toward:

```text
scenarios/
├── methodology/
│   ├── scenario_model/
│   ├── mathematical_methods/
│   ├── normalization/
│   └── decisions/
│
├── generators/
│
├── pools/
│   ├── pool_v1/
│   └── ...
│
├── corpus/
│   └── ...                  # empty until final selection
│
├── manifests/
│
├── analysis/
│   ├── features/
│   ├── similarity/
│   ├── clustering/
│   ├── coverage/
│   ├── figures/
│   └── reports/
│
├── benchmark/
│   ├── routing/
│   ├── reports/
│   └── results/
│
└── _archive/
    └── legacy_methodology/
```

The physical reorganisation of the repository **must not be done automatically** merely because it appears here. This tree defines the architectural direction of the new design.

---

# 12. Source, generated, and historical artefacts

| Class | Description |
|---|---|
| **Scientific source** | schemas, frozen manifests, configurations, generation/selection code, methodological decisions |
| **Generated data** | feature matrices, distances, clusters, figures, regenerable reports |
| **Simulations** | reports produced by The ONE; they may be expensive to regenerate |
| **Legacy** | previous corpus/methodology, kept only as historical reference until a new evaluation |

A legacy artefact must never be presented as evidence of methodology v2 without an explicit decision to reuse it.

---

# 13. Technical guide to configuring The ONE

The following sections keep the useful technical reference from the previous project. Their presence **does not mean that every one of these keys will become a corpus feature**.

Each parameter will later be classified as:

```text
CONSTANT
VARIABLE
DERIVED
ROUTING_ONLY
OUTPUT_ONLY
IMPLEMENTATION_ONLY
```

---

## 13.1 How settings are loaded

- The ONE loads `default_settings.txt` and then the configurations specified for the run.
- Later keys can override earlier values.
- Many options use namespaces such as `Scenario.*`, `MovementModel.*`, `Group.*`, or `Events1.*`.
- Specific values of `Group1.*`, `Group2.*`, and so on make it possible to create heterogeneous groups.
- Basic syntax: `key = value`.
- Comments use `#`.
- Paths must stay portable and documented.
- Numeric values may use suffixes such as `k`, `M`, `G`, `kiB`, `MiB`, `GiB`, depending on the corresponding parser.
- Lists are normally represented as CSV values.

The exact composition policy for overlays/base/scenario/router will be frozen in a later phase.

---

## 13.2 Scenario.*

| Key | Type | Description | Initial methodological status |
|-------|------|-------------|-----------------------------|
| `Scenario.name` | string | Scenario name. | Identifier / metadata |
| `Scenario.endTime` | double (s) | Simulation duration. | Candidate for CONSTANT = 86400 |
| `Scenario.updateInterval` | double (s) | World time step. | Candidate for CONSTANT |
| `Scenario.simulateConnections` | boolean | Enables connections. | Candidate for CONSTANT |
| `Scenario.nrofHostGroups` | int | Number of groups. | Candidate for VARIABLE |

There is no `Simulation.endTime`; `Scenario.endTime` is used.

---

## 13.3 MovementModel.*

| Key | Type | Description | Initial methodological status |
|-------|------|-------------|-----------------------------|
| `MovementModel.worldSize` | CSV | World size. | Candidate for CONSTANT 5000,5000 |
| `MovementModel.rngSeed` | int | RNG seed. | Reproducibility / run dimension |
| `MovementModel.warmup` | double | Warmup. | TBD |

---

## 13.4 MapBasedMovement.*

| Key | Type | Description |
|-------|------|-------------|
| `MapBasedMovement.nrofMapFiles` | int | Number of WKT maps. |
| `MapBasedMovement.mapFile1 ... N` | string | Map files. |

File identity should not be used as a direct numeric feature; map characteristics should be extracted when appropriate.

---

# 14. Groups

Each `GroupN` represents a set of nodes with common behaviour.

| Key | Type | Description |
|-------|------|-------------|
| `groupID` | string | Identifier/prefix. |
| `nrofHosts` | int | Number of hosts. |
| `movementModel` | string | Movement model. |
| `router` | string | Router. **ROUTING_ONLY for corpus identity.** |
| `bufferSize` | numeric | Buffer. |
| `nrofInterfaces` | int | Interfaces. |
| `interface1 ... N` | string | Interface types. |
| `speed` | min,max | Speed. |
| `waitTime` | min,max | Wait. |
| `msgTtl` | int | TTL. |
| `activeTimes` | intervals | Temporal activity. |
| `okMaps` | indices | Allowed maps. |

## 14.1 Group heterogeneity

The methodology must be able to represent scenarios such as:

```text
70 pedestrians
20 cars
10 buses
```

without turning the arbitrary order `Group1/Group2/Group3` into a false mathematical difference.

The vector representation of groups will therefore be a specific decision of Phase 1.

---

# 15. Communication interfaces

Example:

```ini
Group.nrofInterfaces = 1
Group.interface1 = bt0
bt0.type = SimpleBroadcastInterface
bt0.transmitSpeed = 2M
bt0.transmitRange = 10
```

Relevant parameters:

| Key | Meaning |
|---|---|
| `<id>.type` | interface type |
| `transmitSpeed` | transmission speed |
| `transmitRange` | range |
| `scanInterval` | scan interval |

The methodology must decide which of these are constants and which belong to the scenario space.

---

# 16. Movement models

Relevant models known in the project:

| Model | Summary |
|---|---|
| `RandomWaypoint` | random points with movement and waiting |
| `RandomWalk` | random displacement |
| `ShortestPathMapBasedMovement` | lowest-cost routes on a map |
| `MapRouteMovement` | predefined routes |
| `BusMovement` | movement along routes/stops |
| `WorkingDayMovement` | composite daily behaviour |
| `ExternalMovement` | external positions |
| `ExternalPathMovement` | external traces/routes |
| `StationaryMovement` | stationary nodes |
| `LinearMovement` | linear trajectory |
| `ClusterMovement` | cluster-based movement, according to the implementation |

The movement-model class is categorical and needs an explicit representation in the feature space.

---

# 17. WorkingDayMovement

Previous project code and documentation contain useful experience with:

- `HomeActivityMovement`;
- `OfficeActivityMovement`;
- `BusTravellerMovement` / `CarMovement`;
- `EveningActivityMovement`;
- `BusMovement`.

Potentially relevant parameters include:

- `ownCarProb`;
- `probGoShoppingAfterWork`;
- `timeDiffSTD`;
- `workDayLength`;
- location of homes/offices/meeting spots;
- group size;
- activity times.

These variables will not be added to the vector automatically until the semantics of each one are defined.

---

# 18. Events and message traffic

Basic configuration:

```ini
Events.nrof = 1
Events1.class = MessageEventGenerator
Events1.interval = 25, 35
Events1.size = 50k, 150k
Events1.hosts = 0, 100
Events1.prefix = M
```

Relevant parameters:

| Key | Meaning |
|---|---|
| `Events.nrof` | number of generators |
| `Events1.interval` | interval between messages |
| `Events1.size` | size |
| `Events1.hosts` | sender range |
| `Events1.tohosts` | receiver range |
| `Events1.time` | generation window |
| `Events1.prefix` | identifier prefix |

The new model must be able to represent several generators and inter-group traffic.

---

# 19. Routers

Routers that are currently relevant:

| Router | Usual parameters |
|---|---|
| `EpidemicRouter` | — |
| `SprayAndWaitRouter` | `nrofCopies`, `binaryMode` |
| `ProphetRouter` | `secondsInTimeUnit`, `beta`, `gamma` |
| `MaxPropRouter` | specific parameters |
| `FirstContactRouter` | — |
| `DirectDeliveryRouter` | — |
| others / custom | according to the implementation |

**Methodology v2 rule:**

```text
router ∉ scenario feature vector
```

The router belongs to the later benchmark design.

---

# 20. Reports and KPIs

Known reports of interest:

- `MessageStatsReport`;
- `MessageDeliveryReport`;
- `MessageDelayReport`;
- `CreatedMessagesReport`;
- `DeliveredMessagesReport`;
- `ContactTimesReport`;
- `InterContactTimesReport`;
- `TotalContactTimeReport`;
- `BufferOccupancyReport`;
- `MessageCopyCountReport`;
- `DistanceDelayReport`;
- `NodeDensityReport`;
- `LocationSnapshotReport`;
- `MovementNs2Report`.

The final KPI policy must be frozen before the routing benchmark.

Possible target metrics:

```text
delivery probability
latency
overhead ratio
hop count
buffer behaviour
contact statistics
```

Status:

```text
ROUTING_KPI_POLICY = TBD
```

---

# 21. Minimal conceptual example

The following example is technical only; it does not yet define a scientifically valid scenario for the corpus.

```ini
Scenario.name = candidate_001
Scenario.endTime = 86400
Scenario.nrofHostGroups = 1
Scenario.simulateConnections = true
Scenario.updateInterval = 0.1

MovementModel.rngSeed = 1
MovementModel.worldSize = 5000, 5000

Group.movementModel = RandomWaypoint
Group.router = EpidemicRouter
Group.nrofHosts = 40
Group.bufferSize = 5M
Group.nrofInterfaces = 1
Group.interface1 = bt0
Group.speed = 0.5, 1.5
Group.waitTime = 0, 120

bt0.type = SimpleBroadcastInterface
bt0.transmitSpeed = 250k
bt0.transmitRange = 10

Events.nrof = 1
Events1.class = MessageEventGenerator
Events1.interval = 25, 35
Events1.size = 500k, 1M
Events1.hosts = 0, 40
Events1.prefix = M

Report.nrofReports = 2
Report.reportDir = reports/
Report.report1 = MessageStatsReport
Report.report2 = ContactTimesReport
```

For the real corpus, the router should be overridable by an overlay or an equivalent mechanism without changing the identity of the scenario conditions.

---

# 22. Quick namespace reference

| Prefix | Use |
|---------|-----|
| `Scenario.*` | general scenario |
| `MovementModel.*` | world, seed, warmup |
| `MapBasedMovement.*` | maps |
| `Group.*` | group defaults |
| `Group1.*`, `Group2.*`, ... | specific groups |
| `<interface>.*` | communication |
| `Events.*`, `Events1.*`, ... | traffic |
| `Report.*` | outputs |
| `<RouterClass>.*` | routing-specific configuration |

---

# 23. Scientific conventions of the project

## 23.1 Do not confuse parameters with results

Features used to select the corpus must represent scenario conditions, not the performance of a router, unless a future methodological decision explicitly says otherwise.

## 23.2 Do not confuse correlation with linear independence

Pearson and Spearman do not, by themselves, demonstrate linear independence of a set of scenarios.

## 23.3 Do not fix a number of scenarios without a criterion

`N = 50` may be a working budget, but the final corpus must be justified by coverage, saturation, representativeness, and experimental cost.

## 23.4 Do not normalise traces arbitrarily

Every spatio-temporal transformation must declare which quantities it preserves and which it alters.

## 23.5 Do not select the corpus using routing results

The corpus must be independent of the protocol we later want to evaluate.

## 23.6 Separate development, validation, and the final corpus

Scenarios used to tune generation/selection rules must not be treated automatically as final evidence.

---

# 24. Working definitions

**Scenario candidate**  
A valid configuration of experimental conditions that may enter a pool.

**Pool**  
A broad set of candidates before selection.

**Feature vector**  
A normalised, or normalisable, mathematical representation of a scenario.

**Corpus**  
The final frozen set of scenarios selected for the benchmark.

**Routing benchmark**  
Execution of different protocols on exactly the same corpus.

**Coverage**  
The degree to which the corpus represents the defined condition space.

**Redundancy**  
The presence of scenarios that contribute substantially equivalent spatial/parametric information according to the frozen metric.

**Normalization Contract**  
Rules that define how to adapt features and/or physical scenarios while keeping explicit invariants.

---

# 25. Existing implementation that can be reused

The previous repository already contains useful code for:

- batch execution of The ONE;
- parameter/feature extraction;
- normalisation;
- correlations;
- distance matrices;
- figure generation;
- dashboards;
- configuration validation;
- reports.

That code may be reused only after checking that it implements methodology v2.

Do not assume:

```text
existing code == scientific authority
```

Authority will be the frozen methodology and its tests.

---

# 26. Requirements

Approximate technical base:

- Java compatible with the version of The ONE in use;
- The ONE, compiled;
- Python 3;
- `numpy`;
- `pandas`;
- `scipy`;
- `matplotlib`;
- `scikit-learn`, probably required for part of the mathematical analysis/clustering;
- `streamlit`, if the dashboard is kept.

Versions will later be frozen in a reproducible environment (`requirements.txt`, `pyproject.toml`, Conda, or equivalent).

---

# 27. Author / contact

**the-one-scenario-corpus** — methodology and reproducible scenario corpus for benchmarking DTN/OppNet protocols in The ONE.

- **Author:** Raül de Arriba

---
