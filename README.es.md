# the-one-scenario-corpus

Documentos de trabajo: [explicación (EN)](README.md) · [checklist de fases](CHECKLIST.es.md)

## Objetivo del proyecto

**the-one-scenario-corpus** es un proyecto de investigación para diseñar, generar, analizar y seleccionar un **corpus de escenarios reproducibles para The ONE (Opportunistic Network Environment)** que permita comparar protocolos de enrutamiento DTN/OppNet **en igualdad de condiciones experimentales**.

La idea central es separar claramente:

1. las **condiciones del escenario** que describen el entorno en el que ocurre la comunicación;
2. el **protocolo de enrutamiento**, que será la variable experimental que se cambia al comparar algoritmos;
3. los **resultados de simulación**, que se usan únicamente para evaluar el comportamiento del protocolo y no para decidir qué escenarios forman parte del corpus.

Por tanto, un escenario del corpus debe poder ejecutarse repetidamente con distintos routers sin modificar el resto de condiciones:

```text
Scenario_i + EpidemicRouter
Scenario_i + ProphetRouter
Scenario_i + SprayAndWaitRouter
Scenario_i + ...
```

El objetivo final es poder afirmar que las diferencias observadas entre protocolos se deben al comportamiento del protocolo bajo las mismas condiciones y no a haber utilizado escenarios distintos.

---

# Estado metodológico actual

> **ESTADO METODOLÓGICO**

Este proyecto parte de una metodología científica definida desde cero para construir un **corpus reproducible y representativo de escenarios para The ONE Simulator**.

La metodología se estructura alrededor de ocho preguntas fundamentales:

1. qué es exactamente un escenario;
2. qué parámetros se mantienen constantes en todo el corpus;
3. qué parámetros pueden variar entre escenarios;
4. cómo se representa cada escenario mediante un vector de características;
5. cómo se normalizan escenarios de distinta duración, escala o procedencia;
6. cómo se mide similitud, redundancia, diversidad y cobertura entre escenarios;
7. cómo se selecciona un corpus final representativo a partir de un pool de candidatos;
8. cómo se comparan posteriormente distintos protocolos de enrutamiento sobre exactamente las mismas condiciones experimentales.

El desarrollo metodológico seguirá un enfoque incremental: primero se define formalmente el espacio de escenarios y sus características, después se estudian los métodos matemáticos de comparación y selección, y finalmente se construye y valida el corpus.

Estado actual:

```text
METODOLOGÍA = V1 EN DISEÑO
CORPUS FINAL = NO CONSTRUIDO
NÚMERO FINAL DE ESCENARIOS = TBD
FEATURE SPACE = TBD
NORMALIZATION CONTRACT = TBD
SIMILARITY / REDUNDANCY METRICS = TBD
SELECTION / CLUSTERING METHOD = TBD
ROUTING BENCHMARK = NO AUTORIZADO TODAVÍA
```

---

# 1. Pregunta de investigación

La pregunta general del proyecto es:

> **¿Cómo construir un corpus finito de escenarios para The ONE que cubra de forma suficientemente diversa y no redundante el espacio de condiciones relevantes de una red DTN/OppNet, de modo que distintos protocolos de routing puedan compararse sobre exactamente las mismas condiciones experimentales?**

El problema tiene dos partes diferenciadas.

## 1.1 Construcción del corpus

Partimos de un espacio potencialmente muy grande de configuraciones `.settings`.

Necesitamos transformar ese espacio en un conjunto finito que:

- cubra condiciones diferentes;
- evite duplicados y escenarios prácticamente equivalentes;
- preserve casos extremos relevantes;
- sea computacionalmente ejecutable;
- tenga trazabilidad;
- no esté construido para favorecer ningún protocolo concreto.

## 1.2 Benchmark de routing

Una vez congelado el corpus:

```text
Corpus de escenarios
        ↓
Scenario 001 ── Router A / Router B / Router C / ...
Scenario 002 ── Router A / Router B / Router C / ...
...
Scenario N   ── Router A / Router B / Router C / ...
```

Los protocolos deben ejecutarse sobre los mismos escenarios y con las mismas condiciones iniciales siempre que la semántica del simulador lo permita.

---

# 2. Qué entendemos por escenario

En esta metodología, el **routing protocol no forma parte de la identidad del escenario**.

Conceptualmente definimos un escenario como:

\[
S_i = (E, M_i, C_i)
\]

con:

- \(E\): constantes experimentales globales;
- \(M_i\): perfil espacial y de movimiento;
- \(C_i\): perfil de comunicación y tráfico.

El protocolo de routing se aplica posteriormente:

\[
Run_{i,r} = (S_i, R_r)
\]

De este modo, un mismo \(S_i\) puede evaluarse con múltiples \(R_r\).

---

# 3. Descomposición funcional de un `.settings`

Para el proyecto dividimos conceptualmente los parámetros de The ONE en cinco bloques.

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

Son parámetros que queremos fijar globalmente para que los escenarios puedan compararse.

Candidatos iniciales:

```text
MovementModel.worldSize = 5000,5000
Scenario.endTime = 86400        # 24 h
Scenario.updateInterval = TBD
MovementModel.warmup = TBD
seed policy = TBD
```

**Nota:** `MovementModel.worldSize` se expresa en unidades espaciales del simulador, la unidad son metros en los modelos usados por The ONE.

El valor exacto de cada constante la congelaremos únicamente cuando esté justificado.

## 3.2 B. Spatial + Movement profile

Describe dónde están los nodos y cómo se mueven.

Puede incluir:

- mapa o mapas WKT;
- número de nodos;
- grupos de nodos;
- tipo de cada grupo;
- posición inicial;
- modelo de movimiento;
- velocidad;
- tiempos de espera;
- rutas;
- puntos de interés;
- horarios;
- trazas externas;
- parámetros específicos de modelos como `WorkingDayMovement`.

Ejemplos de modelos relevantes:

- `RandomWaypoint`;
- `RandomWalk`;
- `ShortestPathMapBasedMovement`;
- `MapRouteMovement`;
- `BusMovement`;
- `WorkingDayMovement`;
- `ExternalMovement`;
- `ExternalPathMovement`;
- otros disponibles o implementados en el proyecto.

## 3.3 C. Communication + Traffic profile

Describe las oportunidades de comunicación y la carga generada.

### Comunicación

Puede incluir:

- número de interfaces;
- tipo de interfaz;
- rango de transmisión;
- velocidad de transmisión;
- intervalo de escaneo;
- tamaño de buffer;
- TTL.

### Tráfico de mensajes

Puede incluir:

- quién envía;
- quién puede recibir;
- cuándo se generan mensajes;
- intervalo entre mensajes;
- distribución temporal;
- tamaño de los mensajes;
- cantidad/carga total;
- generadores múltiples;
- tráfico entre grupos.

## 3.4 D. Routing protocol

Es la **variable experimental principal del benchmark posterior**.

Ejemplos:

- `EpidemicRouter`;
- `ProphetRouter`;
- `SprayAndWaitRouter`;
- `MaxPropRouter`;
- `FirstContactRouter`;
- `DirectDeliveryRouter`;
- protocolos propios.

La elección del router **no debe utilizarse para decidir qué escenarios forman parte del corpus**.

## 3.5 E. Observation / Reports

Los reports sirven para evaluar el resultado de las simulaciones.

Ejemplos:

- delivery probability;
- latency;
- overhead;
- hop count;
- tiempos de contacto;
- inter-contact times;
- ocupación de buffer;
- copias de mensajes;
- densidad;
- otros KPIs que se congelen para el benchmark.

Estos resultados pertenecen al benchmark de routing y no al proceso inicial de selección del corpus salvo que una futura metodología lo justifique explícitamente.

---

# 4. Representación vectorial del escenario

Cada escenario deberá transformarse en un vector de características:

\[
\mathbf{x}_i = [x_{i1}, x_{i2}, \dots, x_{ip}]
\]

El vector debe representar las condiciones relevantes del escenario **sin incluir la identidad ni los resultados del protocolo de routing**.

La definición exacta de las features todavía está abierta.

## 4.1 Posibles familias de features

### Espacio / mapa

- dimensiones;
- densidad espacial;
- propiedades del grafo viario;
- cobertura;
- longitud de red;
- topología;
- otras propiedades que sean científicamente relevantes.

### Movimiento

- número total de nodos;
- número de grupos;
- proporción de tipos de nodos;
- velocidades;
- wait times;
- modelo de movimiento;
- parámetros de movilidad;
- actividad temporal;
- métricas derivadas de trazas cuando proceda.

### Comunicación

- rango;
- velocidad de transmisión;
- número/tipo de interfaces;
- buffer;
- TTL.

### Tráfico

- tasa de creación de mensajes;
- tamaño;
- distribución temporal;
- distribución origen/destino;
- densidad de carga;
- heterogeneidad entre grupos.

## 4.2 Features numéricas y categóricas

No todas las variables de un `.settings` son numéricas.

Ejemplo:

```text
movementModel = RandomWaypoint
movementModel = WorkingDayMovement
movementModel = ExternalMovement
```

Por tanto, el feature space deberá definir cómo representar:

- variables continuas;
- variables discretas;
- rangos min/max;
- variables categóricas;
- grupos múltiples;
- estructuras jerárquicas.

No se aplicará una codificación final hasta revisar sus consecuencias matemáticas.

---

# 5. Independencia, similitud y redundancia

La metodología v2 debe evitar usar incorrectamente expresiones como:

> "corpus de escenarios linealmente independientes"

sin una definición matemática precisa.

Si los escenarios se representan por vectores de dimensión \(p\), como máximo pueden existir \(p\) vectores linealmente independientes en \(\mathbb{R}^p\).

Por tanto, para un corpus con \(N > p\), **no puede exigirse independencia lineal de todos los escenarios**.

La pregunta metodológica correcta será algo similar a:

> ¿El corpus cubre suficientemente el espacio de características sin contener redundancia innecesaria?

## 5.1 Métodos que deben estudiarse antes de seleccionar uno

La primera fase matemática analizará, como mínimo:

- Pearson;
- Spearman;
- covarianza;
- varianza;
- matriz de correlación;
- distancias Euclídea / Manhattan / cosine y otras justificadas;
- SVD y rango;
- PCA;
- condición numérica;
- nearest-neighbour distance;
- clustering;
- silhouette;
- medidas de coverage;
- criterios de saturación.

Cada método deberá documentarse con:

1. qué pregunta responde;
2. qué supuestos requiere;
3. qué tipos de features admite;
4. qué preprocesamiento necesita;
5. cuándo puede producir conclusiones engañosas;
6. si se utilizará para features, escenarios, clusters o selección.

**No se fijará una métrica únicamente porque ya exista código para calcularla.**

---

# 6. Normalización del feature space

Las features pueden encontrarse en escalas muy diferentes:

```text
nodes = 100
speed = 1.2 m/s
worldSize = 5000 m
messageSize = 1000000 bytes
range = 20 m
endTime = 86400 s
```

Compararlas directamente mediante una distancia produciría resultados dominados por las variables de mayor escala numérica.

La metodología deberá estudiar y congelar un procedimiento de normalización antes del clustering o selección.

Posibles métodos a evaluar:

- min-max;
- z-score;
- robust scaling;
- transformaciones logarítmicas cuando estén justificadas;
- codificación específica para categóricas;
- normalización por magnitudes físicas relevantes.

La normalización será parte del método científico y deberá quedar versionada.

---

# 7. Normalización de escenarios y trazas reales

Además de normalizar vectores, el proyecto necesita estudiar la **normalización física/temporal de escenarios**.

Ejemplo:

```text
Duración estándar del corpus = 24 h
Traza real disponible        = 12 h
```

No se puede simplemente duplicar o escalar la traza sin definir qué propiedad se quiere preservar.

Si transformamos tiempo mediante:

\[
t' = s_t t
\]

y espacio mediante:

\[
\mathbf{p}' = s_x \mathbf{p}
\]

la velocidad efectiva se transforma como:

\[
v' = v \frac{s_x}{s_t}
\]

Por ello se deberá crear un **Normalization Contract** que indique qué invariantes queremos preservar en cada tipo de transformación.

Posibles invariantes:

- velocidad;
- duración;
- densidad espacial;
- distancias relativas;
- tasa de generación de mensajes;
- volumen total de mensajes;
- patrones horarios;
- contactos;
- otros.

Estado:

```text
NORMALIZATION_CONTRACT = TBD
```

No se normalizarán datasets reales de forma definitiva hasta cerrar este contrato.

---

# 8. Generación de pools de escenarios

El nuevo diseño no generará directamente el corpus final.

Primero se crearán **pools de escenarios candidatos**.

```text
Definición del espacio
        ↓
Generación de candidatos
        ↓
Pool de .settings
        ↓
Extracción de features
        ↓
Normalización
        ↓
Análisis matemático
        ↓
Clustering / coverage / redundancy
        ↓
Selección de representantes
        ↓
Corpus final
```

Un pool puede contener muchas más configuraciones que el corpus final.

La generación deberá registrar:

- origen del escenario;
- parámetros efectivos;
- seed;
- reglas de generación;
- versión del generador;
- normalización aplicada;
- hash o identificador reproducible cuando proceda.

---

# 9. Selección del corpus final

El tamaño del corpus final **no se fija todavía**.

```text
TARGET_CORPUS_SIZE = TBD
```

Valores como 50, 70 o 100 pueden estudiarse como presupuestos experimentales, pero no deben convertirse en criterio científico por sí mismos.

La selección final deberá justificar simultáneamente:

- cobertura del espacio;
- separación entre representantes;
- redundancia;
- conservación de extremos;
- representación de clusters;
- estabilidad de la selección;
- coste computacional posterior.

Una posible arquitectura futura es:

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

Los algoritmos concretos permanecen `TBD` hasta completar la revisión matemática.

---

# 10. Igualdad de condiciones en el benchmark

Una vez congelado un escenario \(S_i\), todos los routers deben utilizar el mismo escenario base.

Conceptualmente:

```text
S_i
├── Epidemic
├── Prophet
├── SprayAndWait
├── MaxProp
└── ...
```

La configuración debe diseñarse para que cambiar de router no modifique accidentalmente:

- mapa;
- movilidad;
- nodos;
- tráfico;
- interfaces;
- seeds relevantes;
- tiempo de simulación;
- reports necesarios.

Los parámetros propios de cada router se documentarán por separado.

---

# 11. Estructura recomendada del repositorio

La estructura actual podrá migrarse gradualmente hacia:

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
│   └── ...                  # vacío hasta selección final
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

La reorganización física del repositorio **no debe hacerse automáticamente** únicamente porque aparezca aquí. Este árbol define la dirección arquitectónica del nuevo diseño.

---

# 12. Fuente, generado e histórico

| Clase | Descripción |
|---|---|
| **Fuente científica** | schemas, manifests congelados, configuraciones, código de generación/selección, decisiones metodológicas |
| **Datos generados** | matrices de features, distancias, clusters, figuras, informes regenerables |
| **Simulaciones** | reports generados por The ONE; pueden ser costosos de regenerar |
| **Legacy** | corpus/metodología anterior conservados únicamente como referencia histórica hasta nueva evaluación |

Nunca se debe presentar un artefacto legacy como evidencia de la metodología v2 sin una decisión explícita de reutilización.

---

# 13. Guía técnica de configuración de The ONE

Las siguientes secciones conservan la referencia técnica útil del proyecto anterior. Su presencia **no significa que todas estas claves vayan a convertirse en features del corpus**.

Cada parámetro deberá clasificarse posteriormente como:

```text
CONSTANT
VARIABLE
DERIVED
ROUTING_ONLY
OUTPUT_ONLY
IMPLEMENTATION_ONLY
```

---

## 13.1 Cómo se cargan los settings

- The ONE carga `default_settings.txt` y posteriormente las configuraciones indicadas para la ejecución.
- Las claves posteriores pueden sobrescribir valores anteriores.
- Muchas opciones utilizan namespaces como `Scenario.*`, `MovementModel.*`, `Group.*` o `Events1.*`.
- Los valores específicos de `Group1.*`, `Group2.*`, etc. permiten crear grupos heterogéneos.
- Sintaxis básica: `clave = valor`.
- Comentarios con `#`.
- Las rutas deben mantenerse portables y documentadas.
- Valores numéricos pueden utilizar sufijos como `k`, `M`, `G`, `kiB`, `MiB`, `GiB` según el parser correspondiente.
- Las listas se representan normalmente como valores CSV.

La política exacta de composición por overlays/base/scenario/router deberá congelarse en una fase posterior.

---

## 13.2 Scenario.*

| Clave | Tipo | Descripción | Estado metodológico inicial |
|-------|------|-------------|-----------------------------|
| `Scenario.name` | string | Nombre del escenario. | Identificador / metadata |
| `Scenario.endTime` | double (s) | Duración de simulación. | Candidato a CONSTANT = 86400 |
| `Scenario.updateInterval` | double (s) | Paso temporal del mundo. | Candidato a CONSTANT |
| `Scenario.simulateConnections` | boolean | Activa conexiones. | Candidato a CONSTANT |
| `Scenario.nrofHostGroups` | int | Número de grupos. | Candidato a VARIABLE |

No existe `Simulation.endTime`; se utiliza `Scenario.endTime`.

---

## 13.3 MovementModel.*

| Clave | Tipo | Descripción | Estado metodológico inicial |
|-------|------|-------------|-----------------------------|
| `MovementModel.worldSize` | CSV | Tamaño del mundo. | Candidato a CONSTANT 5000,5000 |
| `MovementModel.rngSeed` | int | Semilla RNG. | Reproducibilidad / run dimension |
| `MovementModel.warmup` | double | Calentamiento. | TBD |

---

## 13.4 MapBasedMovement.*

| Clave | Tipo | Descripción |
|-------|------|-------------|
| `MapBasedMovement.nrofMapFiles` | int | Número de mapas WKT. |
| `MapBasedMovement.mapFile1 ... N` | string | Ficheros de mapa. |

La identidad del fichero no debería usarse como feature numérica directa; deberán extraerse características del mapa cuando proceda.

---

# 14. Grupos

Cada `GroupN` representa un conjunto de nodos con comportamiento común.

| Clave | Tipo | Descripción |
|-------|------|-------------|
| `groupID` | string | Identificador/prefijo. |
| `nrofHosts` | int | Número de hosts. |
| `movementModel` | string | Modelo de movimiento. |
| `router` | string | Router. **ROUTING_ONLY para identidad del corpus.** |
| `bufferSize` | numérico | Buffer. |
| `nrofInterfaces` | int | Interfaces. |
| `interface1 ... N` | string | Tipos de interfaz. |
| `speed` | min,max | Velocidad. |
| `waitTime` | min,max | Espera. |
| `msgTtl` | int | TTL. |
| `activeTimes` | intervalos | Actividad temporal. |
| `okMaps` | índices | Mapas permitidos. |

## 14.1 Heterogeneidad de grupos

La metodología debe poder representar escenarios como:

```text
70 peatones
20 coches
10 buses
```

sin convertir el orden arbitrario `Group1/Group2/Group3` en una falsa diferencia matemática.

Por tanto, la representación vectorial de grupos será una decisión específica de la Fase 1.

---

# 15. Interfaces de comunicación

Ejemplo:

```ini
Group.nrofInterfaces = 1
Group.interface1 = bt0
bt0.type = SimpleBroadcastInterface
bt0.transmitSpeed = 2M
bt0.transmitRange = 10
```

Parámetros relevantes:

| Clave | Significado |
|---|---|
| `<id>.type` | tipo de interfaz |
| `transmitSpeed` | velocidad de transmisión |
| `transmitRange` | alcance |
| `scanInterval` | intervalo de escaneo |

La metodología deberá decidir cuáles son constantes y cuáles forman parte del espacio de escenarios.

---

# 16. Modelos de movimiento

Modelos relevantes conocidos en el proyecto:

| Modelo | Resumen |
|---|---|
| `RandomWaypoint` | puntos aleatorios con movimiento y espera |
| `RandomWalk` | desplazamiento aleatorio |
| `ShortestPathMapBasedMovement` | rutas de menor coste sobre mapa |
| `MapRouteMovement` | rutas predefinidas |
| `BusMovement` | movimiento sobre rutas/paradas |
| `WorkingDayMovement` | comportamiento diario compuesto |
| `ExternalMovement` | posiciones externas |
| `ExternalPathMovement` | trazas/rutas externas |
| `StationaryMovement` | nodos fijos |
| `LinearMovement` | trayectoria lineal |
| `ClusterMovement` | movimiento basado en clusters según implementación |

La clase del movement model es categórica y necesita una representación explícita en el feature space.

---

# 17. WorkingDayMovement

El código y documentación previa del proyecto contienen experiencia útil con:

- `HomeActivityMovement`;
- `OfficeActivityMovement`;
- `BusTravellerMovement` / `CarMovement`;
- `EveningActivityMovement`;
- `BusMovement`.

Parámetros potencialmente relevantes incluyen:

- `ownCarProb`;
- `probGoShoppingAfterWork`;
- `timeDiffSTD`;
- `workDayLength`;
- ubicación de hogares/oficinas/meeting spots;
- tamaño de grupos;
- tiempos de actividad.

Estas variables no se incorporarán automáticamente al vector hasta definir la semántica de cada una.

---

# 18. Eventos y tráfico de mensajes

Configuración básica:

```ini
Events.nrof = 1
Events1.class = MessageEventGenerator
Events1.interval = 25, 35
Events1.size = 50k, 150k
Events1.hosts = 0, 100
Events1.prefix = M
```

Parámetros relevantes:

| Clave | Significado |
|---|---|
| `Events.nrof` | número de generadores |
| `Events1.interval` | intervalo entre mensajes |
| `Events1.size` | tamaño |
| `Events1.hosts` | rango de emisores |
| `Events1.tohosts` | rango de receptores |
| `Events1.time` | ventana de generación |
| `Events1.prefix` | prefijo identificador |

El nuevo modelo deberá ser capaz de representar varios generadores y tráfico entre grupos.

---

# 19. Routers

Routers actualmente relevantes:

| Router | Parámetros habituales |
|---|---|
| `EpidemicRouter` | — |
| `SprayAndWaitRouter` | `nrofCopies`, `binaryMode` |
| `ProphetRouter` | `secondsInTimeUnit`, `beta`, `gamma` |
| `MaxPropRouter` | parámetros específicos |
| `FirstContactRouter` | — |
| `DirectDeliveryRouter` | — |
| otros / propios | según implementación |

**Regla metodológica v2:**

```text
router ∉ feature vector del escenario
```

El router pertenece al diseño de benchmark posterior.

---

# 20. Reports y KPIs

Reports de interés conocidos:

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

La política final de KPIs deberá congelarse antes del benchmark de routing.

Posibles métricas objetivo:

```text
delivery probability
latency
overhead ratio
hop count
buffer behaviour
contact statistics
```

Estado:

```text
ROUTING_KPI_POLICY = TBD
```

---

# 21. Ejemplo mínimo conceptual

El siguiente ejemplo es únicamente técnico; no define todavía un escenario científico válido para el corpus.

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

Para el corpus real, el router debería poder sobrescribirse mediante overlay o mecanismo equivalente sin modificar la identidad de las condiciones del escenario.

---

# 22. Referencia rápida de namespaces

| Prefijo | Uso |
|---------|-----|
| `Scenario.*` | escenario general |
| `MovementModel.*` | mundo, seed, warmup |
| `MapBasedMovement.*` | mapas |
| `Group.*` | defaults de grupo |
| `Group1.*`, `Group2.*`, ... | grupos específicos |
| `<interface>.*` | comunicación |
| `Events.*`, `Events1.*`, ... | tráfico |
| `Report.*` | outputs |
| `<RouterClass>.*` | configuración específica de routing |

---

# 23. Convenciones científicas del proyecto

## 23.1 No confundir parámetros con resultados

Features utilizadas para seleccionar el corpus deben representar condiciones del escenario, no el rendimiento de un router, salvo decisión metodológica explícita futura.

## 23.2 No confundir correlación con independencia lineal

Pearson y Spearman no demuestran por sí solos independencia lineal de un conjunto de escenarios.

## 23.3 No fijar un número de escenarios sin criterio

`N = 50` puede ser un presupuesto de trabajo, pero el corpus final debe justificarse mediante cobertura, saturación, representatividad y coste experimental.

## 23.4 No normalizar trazas de forma arbitraria

Toda transformación espacio-temporal debe declarar qué cantidades conserva y cuáles altera.

## 23.5 No seleccionar el corpus usando resultados de routing

El corpus debe ser independiente del protocolo que posteriormente queremos evaluar.

## 23.6 Separar desarrollo, validación y corpus final

Los escenarios utilizados para ajustar reglas de generación/selección no deben confundirse automáticamente con evidencia final.

---

# 24. Definiciones de trabajo

**Scenario candidate**  
Configuración válida de condiciones experimentales susceptible de entrar en un pool.

**Pool**  
Conjunto amplio de candidatos antes de selección.

**Feature vector**  
Representación matemática normalizada o normalizable de un escenario.

**Corpus**  
Conjunto final congelado de escenarios seleccionados para el benchmark.

**Routing benchmark**  
Ejecución de diferentes protocolos sobre exactamente el mismo corpus.

**Coverage**  
Grado en que el corpus representa el espacio de condiciones definido.

**Redundancy**  
Presencia de escenarios que aportan información espacial/paramétrica sustancialmente equivalente según la métrica congelada.

**Normalization Contract**  
Reglas que definen cómo adaptar features y/o escenarios físicos manteniendo invariantes explícitos.

---

# 25. Implementación existente reutilizable

El repositorio anterior ya contiene código útil para:

- ejecución batch de The ONE;
- extracción de parámetros/features;
- normalización;
- correlaciones;
- matrices de distancia;
- generación de figuras;
- dashboards;
- validación de configuraciones;
- reports.

Ese código puede reutilizarse únicamente después de comprobar que implementa la metodología v2.

No asumir:

```text
existing code == scientific authority
```

La autoridad será la metodología congelada y sus tests.

---

# 26. Requisitos

Base técnica aproximada:

- Java compatible con la versión de The ONE utilizada;
- The ONE compilado;
- Python 3;
- `numpy`;
- `pandas`;
- `scipy`;
- `matplotlib`;
- `scikit-learn` probablemente necesario para parte del análisis matemático/clustering;
- `streamlit` si se mantiene el dashboard.

Las versiones se congelarán posteriormente en un entorno reproducible (`requirements.txt`, `pyproject.toml`, Conda o equivalente).

---

# 27. Autor / contacto

**the-one-scenario-corpus** — metodología y corpus reproducible de escenarios para benchmarking de protocolos DTN/OppNet en The ONE.

- **Autor:** Raül de Arriba

---

