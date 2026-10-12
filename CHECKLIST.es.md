# Checklist metodológico

Documentos de trabajo: [explicación (ES)](README.es.md) · [explanation (EN)](README.md)

El estado de las fases, las preguntas abiertas y la regla de trabajo viven en este documento. Las definiciones del corpus están en los README.

---

# Roadmap metodológico v2

## Fase 0 — Reboot y documentación

**Objetivo:** establecer el nuevo modelo conceptual y eliminar ambigüedades heredadas.

- [x] Reiniciar metodología.
- [x] Separar escenario y routing protocol.
- [x] Documentar estructura conceptual del `.settings`.
- [ ] Congelar glosario.
- [ ] Congelar definición formal de escenario.

## Fase 1 — Scenario Model & Feature Space Definition

**Objetivo:** definir exactamente qué variables describen un escenario.

- [ ] Inventariar claves relevantes de The ONE.
- [ ] Clasificar: CONSTANT / VARIABLE / DERIVED / ROUTING_ONLY / OUTPUT_ONLY.
- [ ] Definir familias de features.
- [ ] Definir tratamiento de grupos múltiples.
- [ ] Definir tratamiento de categóricas.
- [ ] Crear schema de escenario.

**Salida esperada:** `scenario_schema_v1`.

## Fase 2 — Mathematical Diversity / Redundancy Study

**Objetivo:** estudiar matemáticamente cómo comparar escenarios representados como vectores.

- [ ] Pearson.
- [ ] Spearman.
- [ ] Varianza/covarianza.
- [ ] Distancias.
- [ ] Rank/SVD.
- [ ] PCA.
- [ ] Clustering.
- [ ] Coverage/saturation.
- [ ] Seleccionar y justificar el método final.

**Salida esperada:** protocolo matemático preregistrado.

## Fase 3 — Normalization Contract

**Objetivo:** definir normalización matemática y física.

- [ ] Escalado de features.
- [ ] Variables categóricas.
- [ ] Duraciones diferentes.
- [ ] Escalas espaciales diferentes.
- [ ] Trazas externas.
- [ ] Message load.
- [ ] Invariantes que se preservan.

## Fase 4 — Candidate Space Definition

**Objetivo:** definir qué parámetros pueden variar y sus dominios.

- [ ] Movement.
- [ ] Node population.
- [ ] Communication.
- [ ] Traffic.
- [ ] Spatial/map configuration.
- [ ] Constraints entre parámetros.

## Fase 5 — Pool Generation

**Objetivo:** generar configuraciones candidatas reproducibles.

- [ ] Generador de `.settings`.
- [ ] Manifest.
- [ ] Provenance.
- [ ] Validación sintáctica.
- [ ] Validación de ejecución The ONE.

## Fase 6 — Feature Extraction

**Objetivo:** convertir cada candidato en su vector \(\mathbf{x}_i\).

- [ ] Extractor determinista.
- [ ] Tests.
- [ ] Matriz de features.
- [ ] Normalización congelada.

## Fase 7 — Pool Analysis & Clustering

**Objetivo:** medir diversidad, redundancia y estructura del pool.

- [ ] Matriz de distancias/similitudes.
- [ ] Clustering.
- [ ] Estabilidad.
- [ ] Coverage.
- [ ] Saturación.

## Fase 8 — Corpus Selection & Freeze

**Objetivo:** seleccionar los representantes finales.

- [ ] Regla de selección.
- [ ] Número final justificado.
- [ ] Manifest final.
- [ ] Hash/freeze.

## Fase 9 — Routing Benchmark

**Objetivo:** ejecutar routers sobre el corpus congelado.

- [ ] Router matrix.
- [ ] Seeds/runs.
- [ ] Reports.
- [ ] KPI policy.
- [ ] Statistical comparison.

## Fase 10 — Thesis / Paper

- [ ] Metodología.
- [ ] Validación.
- [ ] Resultados.
- [ ] Threats to validity.
- [ ] Reproducibility package.

---

# Preguntas abiertas

Las siguientes cuestiones deben resolverse explícitamente durante las primeras fases:

- [ ] ¿`5000 x 5000 m` será el world size definitivo?
- [ ] ¿24 h será siempre la duración estándar?
- [ ] ¿Qué `updateInterval` se utilizará?
- [ ] ¿Cuál será la política de seeds/runs?
- [ ] ¿El mapa variará en todos los pools o habrá experimentos por capas?
- [ ] ¿Qué movement models entran en el dominio científico?
- [ ] ¿Qué parámetros de comunicación serán constantes?
- [ ] ¿Qué parámetros de tráfico serán variables?
- [ ] ¿Cómo se representan múltiples grupos?
- [ ] ¿Cómo se codifican variables categóricas?
- [ ] ¿Qué features derivadas necesitamos?
- [ ] ¿Qué normalización matemática se adopta?
- [ ] ¿Qué normalización física se adopta para trazas reales?
- [ ] ¿Qué métricas matemáticas medirán redundancia/cobertura?
- [ ] ¿Qué clustering se utilizará, si alguno?
- [ ] ¿Cómo se seleccionará un representante de cada cluster?
- [ ] ¿Qué criterio determina el tamaño final del corpus?
- [ ] ¿Qué routers forman el benchmark inicial?
- [ ] ¿Qué KPIs compararán los routers?

---

# Regla de trabajo

Este README actúa como **documento principal de contexto del reboot metodológico**.

Cada nueva fase deberá:

1. partir de las definiciones aquí descritas;
2. convertir los `TBD` relevantes en decisiones justificadas;
3. documentar cualquier cambio de modelo;
4. no declarar un corpus final antes de cerrar el feature space y la metodología de selección;
5. mantener separado el diseño del corpus del benchmark de protocolos.

La siguiente fase recomendada es:

```text
PHASE 1 — SCENARIO MODEL & FEATURE SPACE DEFINITION
```

Objetivo inmediato:

> crear una clasificación completa de los parámetros `.settings` de The ONE y decidir cuáles son constantes, variables del escenario, features derivadas, parámetros exclusivos del router y parámetros exclusivamente observacionales.
