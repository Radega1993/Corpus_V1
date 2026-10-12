# Estudio de métodos para diversidad, redundancia y selección de escenarios DTN/OppNet

**Proyecto:** corpus reproducible de escenarios para The ONE.  
**Fecha:** 9 de octubre de 2026.  
**Última actualización:** 12 de octubre de 2026.  
**Versión del documento:** 1.1.  
**Estado:** revisión conceptual y propuesta pendiente de validación y preregistro.

## 1. Objetivo y alcance

Este documento desarrolla el punto 5.1 de la metodología: estudiar qué aporta cada método antes de elegir una representación, una distancia o un algoritmo de selección.

La pregunta central es:

> ¿Cómo seleccionar un conjunto finito de escenarios que represente el dominio definido, conserve condiciones relevantes y evite redundancia innecesaria, sin utilizar el rendimiento de ningún protocolo de routing?

Se documentan los seis aspectos solicitados para cada método, con fórmulas, ejemplos y referencias académicas. Las aplicaciones a The ONE y la propuesta final son **razonamiento metodológico para este proyecto**, no resultados ya demostrados por los artículos citados.

Esta revisión puede realizarse antes de cerrar el schema. **Aplicar los métodos a datos y congelar decisiones exige definir primero las features, su semántica y el dominio de escenarios.** No se recuperan como autoridad las features del trabajo legacy.

### 1.1 Tres preguntas distintas

| Pregunta | Objeto analizado | Métodos relevantes |
|---|---|---|
| ¿Las features varían, se solapan o generan dependencias? | Columnas de la matriz | Varianza, covarianza, Pearson, Spearman, SVD, PCA, condición |
| ¿Dos escenarios se parecen bajo la representación elegida? | Filas de la matriz | Distancias, disimilitudes, vecinos próximos |
| ¿El corpus representa el dominio con suficiente detalle? | Subconjunto respecto a una referencia | Coverage, selección, saturación, evaluación por estratos |

Una correlación baja entre columnas no demuestra que las filas sean distintas. Una separación alta entre representantes tampoco demuestra que no haya grandes regiones sin representar.

### 1.2 Escenario, realización y resultado

Se mantiene la definición de trabajo:

$$
S_i=(E,M_i,C_i), \qquad Run_{i,r,k}=(S_i,R_r,\omega_k).
$$

- \(E\): condiciones globales.
- \(M_i\): condiciones espaciales y de movilidad.
- \(C_i\): comunicación y tráfico.
- \(R_r\): routing, fuera de la identidad del escenario.
- \(\omega_k\): realización estocástica, registrada para reproducibilidad.

El vector no incluye delivery probability, latency ni overhead de un router. Una nueva seed no se considera automáticamente un escenario nuevo. Si se incorporan descriptores derivados de movilidad o contactos, habrá que justificar su independencia del routing y cuantificar su variabilidad entre realizaciones.

### 1.3 Implementación de referencia: The ONE 1.6.0

**Decisión del proyecto, 12 de octubre de 2026:** utilizar **The ONE 1.6.0 del repositorio akeranen/the-one** como base del inventario de settings y de la metodología.

La [página oficial de The ONE](https://akeranen.github.io/the-one/) enlaza la versión [v1.6.0](https://github.com/akeranen/the-one/tree/v1.6.0). Se ha obtenido una copia de esa etiqueta y verificado su commit y el hash del fichero de configuración distribuido.

| Campo | Valor o estado |
|---|---|
| Repositorio de referencia | [akeranen/the-one](https://github.com/akeranen/the-one) |
| Versión seleccionada | 1.6.0 |
| Etiqueta verificada | v1.6.0 |
| Commit de la etiqueta | 42b6e20f45799f2533866df215768fa9aedd0f4f |
| SHA-256 de default_settings.txt de la etiqueta | 9e8cceda868bfce709946b31304352dc0b1e8369f553e31f9dbe12795ca383b5 |
| Modificaciones o extensiones de la instalación del proyecto | TBD: no se ha inspeccionado la instalación del usuario |
| Versión de Java y entorno de ejecución | TBD: pendiente de registrar y validar |

El hash se ha calculado sobre los **bytes originales del archivo recuperado de la etiqueta**, sin normalizar finales de línea. No se atribuye ese hash al archivo local del usuario ni a futuros overlays.

**Consecuencia para el estudio:** las claves, defaults, unidades, modelos y parámetros derivados deberán justificarse mediante el código de esta revisión. La rama master y la documentación de otras revisiones no se usarán como sustitutos del inventario de v1.6.0. La versión del simulador no decide por sí sola las features ni la métrica matemática.

La base upstream queda identificada. El cierre completo del ámbito de implementación requiere registrar también las extensiones, parches y entorno que realmente se utilicen. Las constantes del corpus, la normalización y la selección siguen pendientes de definición.

## 2. Notación y condiciones comunes

Sea \(P=\{S_1,\ldots,S_N\}\) un pool de candidatos válidos. La representación produce una matriz:

$$
X\in\mathbb{R}^{N\times p}, \qquad X_{ij}=x_{ij}.
$$

Esta notación real se aplica a un bloque numérico o a una codificación justificada; un escenario puede conservar además categorías y estructuras.

- Una **fila** representa un escenario.
- Una **columna** representa una feature.
- \(Z\) es el bloque numérico transformado, escalado y, cuando corresponda, centrado.
- \(C\subseteq P\) es el corpus seleccionado.
- \(V\) es la referencia de monitorización utilizada durante la selección.
- \(H\) es un conjunto reservado de evaluación, con una política de generación documentada.
- \(d(x,y)\) es la distancia o disimilitud congelada.

Antes de calcular:

1. Resolver defaults y overlays para obtener parámetros efectivos.
2. Mantener invariancia al renombrado de grupos **junto con sus referencias de tráfico, interfaces y rutas**. Reordenar grupos sin actualizar direcciones no garantiza equivalencia.
3. Separar constantes, valores ausentes y parámetros no aplicables.
4. Registrar unidades, transformaciones, dominios y pesos.
5. Definir cómo se representan grupos y generadores múltiples.
6. Distinguir equivalencia de configuración y equivalencia de representación.

**Dos configuraciones distintas pueden tener el mismo vector.** Una distancia sobre vectores será entonces una pseudodistancia sobre los escenarios originales: \(d=0\) no demuestra igualdad física ni igualdad de rendimiento.

Para describir un pool finito no es necesario asumir que los candidatos sean observaciones aleatorias independientes. Para intervalos de confianza, pruebas de hipótesis o generalización sí hay que justificar el muestreo. Un grid sintético no se convierte en muestra representativa por contener muchos puntos.

## 3. Fichas de los métodos

### 3.1 Pearson

**1. Qué pregunta responde.** ¿Dos features cambian conjuntamente de manera lineal entre los escenarios?

$$
r_{ab}=
\frac{\sum_i(x_{ia}-\bar x_a)(x_{ib}-\bar x_b)}
{\sqrt{\sum_i(x_{ia}-\bar x_a)^2}
 \sqrt{\sum_i(x_{ib}-\bar x_b)^2}}.
$$

**2. Supuestos.** Ambas variables deben variar. Calcular el coeficiente no exige normalidad; interpretar una asociación lineal requiere examinar la relación. La inferencia clásica añade supuestos que no presuponemos en el pool.

**3. Features admitidas.** Numéricas con diferencias interpretables. No asignar números arbitrarios a modelos de movilidad nominales.

**4. Preprocesamiento.** Alinear observaciones y documentar ausencias y outliers. Un escalado positivo de unidades no cambia \(r\); una transformación logarítmica puede cambiarlo.

**5. Conclusiones engañosas.** \(r\approx0\) no excluye dependencia no lineal. Una correlación alta puede ser consecuencia del generador o de mezclar poblaciones. No eliminar automáticamente una feature con significado propio.

**6. Uso propuesto.** Diagnóstico de **features**, global y por estratos. No usar Pearson entre filas con magnitudes heterogéneas para decidir redundancia.

**Ejemplo.** Con área constante \(A\), densidad \(N/A\) y número de nodos son dependientes por construcción.

**Paper:** Pearson (1895), [R1]. Referencia histórica; no establece un umbral de descarte para nuestro corpus.

### 3.2 Spearman

**1. Qué pregunta responde.** ¿Dos features mantienen una relación monótona, aunque no sea lineal?

$$
\rho_{ab}=\operatorname{corr}
\left(\operatorname{rank}(X_{\cdot a}),
      \operatorname{rank}(X_{\cdot b})\right).
$$

**2. Supuestos.** Orden interpretable y variación de rangos. No exige normalidad para calcularlo. Los empates necesitan una política explícita.

**3. Features admitidas.** Numéricas y ordinales justificadas; no categorías nominales.

**4. Preprocesamiento.** Usar rangos medios para empates y documentar ausencias. No necesita z-score.

**5. Conclusiones engañosas.** Detecta tendencia monótona, no cualquier dependencia. Muchos empates reducen resolución. No mide la magnitud física de las diferencias.

**6. Uso propuesto.** Diagnóstico de **features**, complementario a Pearson; no selección directa.

**Ejemplo.** Dos medidas del tamaño de una red pueden ordenar de manera parecida los mapas sin tener una relación lineal.

**Paper:** Spearman (1904), [R2]. Fundamenta asociación mediante rangos.

### 3.3 Covarianza

**1. Qué pregunta responde.** ¿Cómo varían conjuntamente dos features y con qué magnitud en sus unidades?

$$
\widehat{\operatorname{cov}}(a,b)=
\frac{1}{N-1}\sum_i(x_{ia}-\bar x_a)(x_{ib}-\bar x_b).
$$

**2. Supuestos.** Datos numéricos y \(N\ge2\) para esta fórmula. Su interpretación poblacional requiere momentos segundos finitos y muestreo adecuado.

**3. Features admitidas.** Numéricas. Su unidad es el producto de las unidades originales.

**4. Preprocesamiento.** Centrar y revisar unidades. Emplear acumulación numéricamente estable, especialmente en pools grandes.

**5. Conclusiones engañosas.** Una covarianza grande puede reflejar unidades grandes. Covarianza cero no implica independencia. La matriz puede ser singular por dependencias exactas.

**6. Uso propuesto.** Diagnóstico de **features** y fundamento de PCA y Mahalanobis.

**Ejemplo.** Cambiar velocidades de m/s a otra unidad altera covarianzas sin modificar escenarios.

**Paper:** Welford (1962), [R3], sobre sumas corregidas de cuadrados y productos. Complemento matricial: Ledoit y Wolf (2004), [R4].

### 3.4 Varianza

**1. Qué pregunta responde.** ¿Cuánto cambia una feature dentro del pool?

$$
s_j^2=\frac{1}{N-1}\sum_i(x_{ij}-\bar x_j)^2.
$$

**2. Supuestos.** Datos numéricos y \(N\ge2\). Como resumen descriptivo no requiere normalidad. Para describir la distribución uniforme del pool completo también puede usarse divisor \(N\), declarando la convención.

**3. Features admitidas.** Numéricas; ordinales solo si las separaciones tienen significado.

**4. Preprocesamiento.** Examinar primero unidades originales. Tras z-score, las columnas no constantes tienen varianza unitaria: ya no detectamos así su escasa variación original.

**5. Conclusiones engañosas.** Baja varianza no significa irrelevancia: puede revelar un pool que no explora esa dimensión. Una categoría rara importante no debe descartarse automáticamente.

**6. Uso propuesto.** Control de **features**: constantes, errores y variación insuficiente.

**Ejemplo.** Un TTL fijado globalmente se conserva en metadata, pero no separa candidatos.

**Paper:** Welford (1962), [R3]. Trata el cálculo estable; la relevancia científica se decide aparte.

### 3.5 Matriz de correlación

**1. Qué pregunta responde.** ¿Qué patrón de asociaciones existe entre todas las features?

$$
R_{ab}=\operatorname{corr}(X_{\cdot a},X_{\cdot b}), \qquad
R=D^{-1/2}\widehat\Sigma D^{-1/2},
$$

donde \(D\) contiene las varianzas de columnas no constantes. Spearman utiliza correlaciones de rangos.

**2. Supuestos.** Los de cada coeficiente. Conviene utilizar las mismas observaciones para todas las parejas.

**3. Features admitidas.** Numéricas para Pearson; numéricas u ordinales para Spearman.

**4. Preprocesamiento.** Excluir constantes del cálculo, registrar transformaciones y analizar globalmente y por estratos. No convertir categorías nominales en números arbitrarios.

**5. Conclusiones engañosas.** La eliminación de ausencias por parejas puede generar una matriz no semidefinida positiva. Un heatmap sin correlaciones extremas no excluye dependencia multivariante: \(x_3=x_1+x_2\) puede ser exacta sin correlaciones por parejas próximas a uno.

**6. Uso propuesto.** Auditoría de **features**, complementada con SVD y revisión semántica.

**Papers:** Pearson [R1] y Spearman [R2] fundamentan sus entradas. Ledoit y Wolf [R4] estudian estimación de covarianza y condicionamiento; no nuestra regla de depuración.

### 3.6 Distancia Euclídea

**1. Qué pregunta responde.** ¿Qué separación tienen dos escenarios en una geometría numérica cuadrática?

$$
d_2(z_i,z_\ell)=
\sqrt{\sum_{j=1}^{p}w_j(z_{ij}-z_{\ell j})^2},
\qquad w_j>0.
$$

**2. Supuestos.** Coordenadas comparables y diferencias interpretables. Los pesos determinan importancia; pesos nulos pueden producir una pseudométrica sobre el espacio original.

**3. Features admitidas.** Numéricas; categorías únicamente con codificación y ponderación justificadas.

**4. Preprocesamiento.** Transformación justificada, escalado congelado y pesos por familias.

**5. Conclusiones engañosas.** Las diferencias grandes pesan mucho. Features redundantes cuentan varias veces. Una distancia matemática no garantiza equivalencia científica.

**6. Uso propuesto.** Comparación de **escenarios**; baseline para vecinos, clustering y cobertura.

**Ejemplo.** Sin escalado, bytes de mensaje pueden dominar frente a m/s.

**Paper:** Aggarwal, Hinneburg y Keim (2001), [R5], sobre normas en alta dimensión. No sustituye validación en nuestro dominio.

### 3.7 Distancia Manhattan

**1. Qué pregunta responde.** ¿Cuál es la suma de diferencias absolutas entre escenarios?

$$
d_1(z_i,z_\ell)=
\sum_{j=1}^{p}w_j|z_{ij}-z_{\ell j}|,\qquad w_j>0.
$$

**2. Supuestos.** Diferencias comparables y significado aditivo. Varias diferencias pequeñas pueden compensar una grande en el valor agregado.

**3. Features admitidas.** Numéricas; otras codificaciones solo con semántica declarada.

**4. Preprocesamiento.** Escalado y pesos por familias, evitando dominio por número de columnas.

**5. Conclusiones engañosas.** No elimina concentración de distancias. Menor penalización cuadrática no implica inmunidad a outliers ni relevancia de las features.

**6. Uso propuesto.** Comparación de **escenarios**, alternativa a Euclídea y componente numérico de la propuesta mixta.

**Paper:** Aggarwal et al. (2001), [R5]. Estudia \(L_1\) y \(L_2\); no demuestra superioridad universal de Manhattan.

### 3.8 Coseno

**1. Qué pregunta responde.** ¿Dos vectores tienen una orientación parecida, ignorando su magnitud?

$$
\operatorname{sim}_{\cos}(x,y)=
\frac{x^\top y}{\|x\|_2\|y\|_2},
\qquad d_{\cos}=1-\operatorname{sim}_{\cos}.
$$

**2. Supuestos.** Vectores no nulos y origen significativo. Ignorar magnitud debe responder al objetivo científico.

**3. Features admitidas.** Vectores numéricos, especialmente perfiles cuya dirección interese.

**4. Preprocesamiento.** Definir transformaciones y pesos. Centrar cambia el origen. Normalizar por norma elimina deliberadamente magnitud.

**5. Conclusiones engañosas.** \(x\) y \(10x\) tienen coseno uno, aunque representen cargas muy distintas. \(1-\cos\) no satisface en general la desigualdad triangular ni distingue vectores proporcionales.

**6. Uso propuesto.** Análisis auxiliar de **perfiles**; no distancia principal de magnitudes absolutas.

**Paper:** Salton, Wong y Yang (1975), [R6], sobre modelo vectorial en recuperación de información. Su aplicación a DTN requiere justificación propia.

### 3.9 Gower para features mixtas

**1. Qué pregunta responde.** ¿Qué disimilitud tienen escenarios con variables numéricas y categóricas?

Para un caso completo con pesos fijos:

$$
d_G(i,\ell)=\frac{\sum_jw_j\delta_j(i,\ell)}{\sum_jw_j}.
$$

Para una numérica, \(\delta_j=|x_{ij}-x_{\ell j}|/R_j\). Para una nominal, cero si coincide y uno si difiere.

**2. Supuestos.** Tipos correctos, rangos positivos, pesos y tratamiento de ausencias definidos. Diferencias entre categorías pueden necesitar consideración científica adicional.

**3. Features admitidas.** Numéricas y nominales; ordinales y binarias con convenciones explícitas.

**4. Preprocesamiento.** Fijar rangos del dominio o desarrollo; separar constantes; evitar rangos recalculados con cada pool. La adaptación por familias es una decisión adicional del proyecto.

**5. Conclusiones engañosas.** Muchas coincidencias pueden diluir una diferencia crítica. Si ausencias o no aplicabilidad cambian el denominador según la pareja, no asumir propiedades métricas automáticamente.

**6. Uso propuesto.** Comparación de **escenarios**, vecinos, clustering compatible y cobertura. Candidata principal mixta.

**Paper:** Gower (1971), [R7]. Presenta similitud para tipos diferentes; estudiamos su disimilitud y una adaptación declarada.

### 3.10 Mahalanobis

**1. Qué pregunta responde.** ¿Qué separación tienen dos vectores considerando covarianzas?

$$
d_M(x,y)=\sqrt{(x-y)^\top\widehat\Sigma^{-1}(x-y)}.
$$

**2. Supuestos.** Covarianza invertible y estable. La fórmula no exige normalidad; interpretaciones probabilísticas añaden supuestos.

**3. Features admitidas.** Numéricas; categorías codificadas pueden introducir dependencias y una geometría difícil de defender.

**4. Preprocesamiento.** Revisar transformaciones, dependencias y singularidad. Considerar regularización, declarando que modifica geometría.

**5. Conclusiones engañosas.** La covarianza depende del muestreo del generador. Las direcciones con escasa varianza pueden amplificarse mucho. La pseudoinversa ignora ciertas direcciones; no recupera información.

**6. Uso propuesto.** Sensibilidad sobre **escenarios numéricos**, no opción principal inicial.

**Papers:** Mahalanobis (1936), [R8], disponible en reimpresión editorial; Ledoit y Wolf (2004), [R4], sobre regularización de covarianza.

### 3.11 SVD y rango

**1. Qué pregunta responde.** ¿Cuántas direcciones lineales contiene la matriz y qué dependencias aparecen?

$$
Z=U\Sigma V^\top,\qquad
\operatorname{rank}(Z)=\#\{\sigma_j>0\}.
$$

En cálculo numérico se declara una tolerancia, por ejemplo:

$$
r_\tau=\#\{\sigma_j>\tau\sigma_1\}.
$$

**2. Supuestos.** Matriz numérica finita. La interpretación depende de centrado, escalado y umbral.

**3. Features admitidas.** Numéricas o codificaciones justificadas. One-hot completo y proporciones que suman uno tienen restricciones conocidas.

**4. Preprocesamiento.** Revisar relaciones analíticas y constantes; congelar escalado. Calcular SVD sin formar innecesariamente \(Z^\top Z\).

**5. Conclusiones engañosas.** Rango completo no prueba independencia estadística ni ausencia de redundancia entre escenarios. Para \(Z\) centrada, \(\operatorname{rank}(Z)\le\min(N-1,p)\). Tener \(N>p\) no es un defecto.

**6. Uso propuesto.** Auditoría de **features y representación**. No limitar el corpus a \(p\) escenarios.

**Paper:** Golub y Reinsch (1970), [R9], referencia clásica de SVD y mínimos cuadrados.

### 3.12 PCA

**1. Qué pregunta responde.** ¿Qué combinaciones lineales explican mayor variación y cómo visualizarla o comprimirla?

Para SVD de \(Z\) centrada:

$$
\lambda_j=\frac{\sigma_j^2}{N-1},
\qquad EVR_j=\frac{\lambda_j}{\sum_\ell\lambda_\ell}.
$$

**2. Supuestos.** La geometría lineal y el criterio de varianza deben ser útiles. Su cálculo no necesita normalidad.

**3. Features admitidas.** Numéricas. Codificar categorías no convierte PCA en solución automática al espacio mixto.

**4. Preprocesamiento.** Centrar; elegir covarianza o variables estandarizadas; documentar transformaciones y outliers.

**5. Conclusiones engañosas.** Varianza no equivale a importancia científica. La proyección puede ocultar diferencias raras. Explicar 95% de varianza no garantiza conservar vecinos, extremos ni cobertura.

**6. Uso propuesto.** Exploración de **features y escenarios**. Inicialmente visualización; selección proyectada solo tras validar pérdida de información.

**Papers:** Pearson (1901), [R10], antecedente original; Jolliffe y Cadima (2016), [R11], revisión complementaria.

### 3.13 Condición numérica

**1. Qué pregunta responde.** ¿Cuán sensible es un cálculo a pequeñas perturbaciones?

Para un bloque \(Z\) de rango columna completo:

$$
\kappa_2(Z)=\frac{\sigma_{\max}}{\sigma_{\min}}.
$$

**2. Supuestos.** Definir problema y matriz. Si falta rango columna, no hay inversión estable de esa representación: reportar deficiencia en lugar de ocultarla mediante truncamiento.

**3. Features admitidas.** Matrices numéricas de PCA, regresión, whitening o Mahalanobis.

**4. Preprocesamiento.** Retirar constantes del bloque centrado y registrar escala y tolerancia. Un ratio sobre singular values retenidos se etiqueta como condición del subespacio retenido.

**5. Conclusiones engañosas.** Depende de unidades y codificaciones. Mala condición no implica corpus poco diverso. En rango columna completo, \(\kappa_2(Z^\top Z)=\kappa_2(Z)^2\).

**6. Uso propuesto.** Control de **representación y cálculos**, antes de invertir covarianzas.

**Paper:** Rice (1966), [R12], teoría del condicionamiento; no proporciona un umbral universal de calidad de corpus.

### 3.14 Nearest-neighbour distance

**1. Qué pregunta responde.** ¿Cuál es el escenario más parecido a cada candidato?

$$
\nu_i=\min_{\ell\ne i}d(x_i,x_\ell).
$$

Para cobertura, la pregunta diferente es:

$$
r(x,C)=\min_{c\in C}d(x,c).
$$

**2. Supuestos.** Distancia congelada, exclusión del propio punto y proximidad discriminativa.

**3. Features admitidas.** Las de la distancia, incluidas mixtas si el cálculo es compatible.

**4. Preprocesamiento.** Separar duplicados exactos de proximidad. En búsqueda aproximada, cuantificar error y verificar casos críticos exactamente.

**5. Conclusiones engañosas.** Proximidad no implica equivalencia física. \(d\le\varepsilon\) no es transitiva: cadenas de puntos cercanos pueden tener extremos lejanos. En alta dimensión las distancias pueden concentrarse.

**6. Uso propuesto.** **Redundancia local, escenarios aislados y cobertura.**

**Paper:** Beyer et al. (1999), [R13]. Estudia cuándo el vecino próximo pierde capacidad discriminativa; no afirma inutilidad universal en alta dimensión.

### 3.15 Clustering

**1. Qué pregunta responde.** ¿Existen grupos relativamente similares bajo una representación y objetivo concretos?

$$
\text{k-means: }\min\sum_i\|z_i-\mu_{g(i)}\|_2^2,
$$

$$
\text{k-medoids: }\min_{\{m_1,\ldots,m_K\}\subseteq P}
\sum_i\min_k d(x_i,x_{m_k}).
$$

**2. Supuestos.** Dependen del algoritmo: k-means usa un objetivo Euclídeo cuadrático; k-medoids usa representantes observados; métodos de densidad requieren decisiones de vecindad y densidad.

**3. Features admitidas.** k-means: numéricas con geometría Euclídea. k-medoids: disimilitudes compatibles. Métodos jerárquicos: depende del linkage.

**4. Preprocesamiento.** Congelar distancia, parámetros, inicialización y ruido. Ward no se aplica sin justificación a una matriz Gower arbitraria.

**5. Conclusiones engañosas.** Se puede particionar un continuo sin clusters naturales. Grupos visualmente claros no prueban representatividad. Un representante por cluster puede perder extremos.

**6. Uso propuesto.** **Estructura y clusters**; selección auxiliar. No es obligatorio agrupar para seleccionar.

**Paper:** Jain (2010), [R14], revisión metodológica. Para estabilidad: Hennig (2007), [R18].

### 3.16 Silhouette

**1. Qué pregunta responde.** ¿Un punto queda más próximo a su cluster que a los alternativos?

Sea \(a(i)\) la distancia media al resto de su cluster y \(b(i)\) la menor distancia media a otro cluster:

$$
s(i)=\frac{b(i)-a(i)}{\max\{a(i),b(i)\}}.
$$

**2. Supuestos.** Al menos dos clusters y disimilitudes coherentes. Singletons se puntúan convencionalmente con cero; declarar tratamiento de denominador cero.

**3. Features admitidas.** Las de la distancia utilizada.

**4. Preprocesamiento.** Misma geometría que el clustering; informar valores por cluster, tamaños, singletons y ruido. Registrar estimaciones basadas en muestras.

**5. Conclusiones engañosas.** Favorece ciertas geometrías compactas. La media oculta grupos problemáticos. Maximizarla no prueba el número verdadero de grupos ni determina tamaño del corpus.

**6. Uso propuesto.** Validación auxiliar de **clusters**, no objetivo principal de selección o parada.

**Paper:** Rousseeuw (1987), [R15], artículo original del método.

### 3.17 Coverage

**1. Qué pregunta responde.** ¿Hasta qué punto una referencia queda representada por el corpus?

$$
R_{\max}(C;H)=\max_{x\in H}r(x,C),
\qquad R_{95}(C;H)=Q_{0.95}\{r(x,C):x\in H\},
$$

$$
\operatorname{Cov}_{\varepsilon}(C;H)=
\frac{\sum_{x\in H}a_x\mathbf{1}[r(x,C)\le\varepsilon]}
{\sum_{x\in H}a_x}, \qquad a_x>0.
$$

**2. Supuestos.** Distancia, referencia, tolerancia y pesos definidos. Para cobertura poblacional, justificar cómo la referencia representa esa población.

**3. Features admitidas.** Las de la distancia; añadir comprobaciones de categorías e interacciones obligatorias.

**4. Preprocesamiento.** Congelar referencia, pesos y tolerancias antes de evaluación final. Informar estratos y puntos peor cubiertos.

**5. Conclusiones engañosas.** Cubrir el pool no demuestra cubrir regiones omitidas por el generador. La media oculta huecos. Los cuantiles dependen de la distribución; el máximo, de sus extremos.

**6. Uso propuesto.** Criterio central de **evaluación y selección**.

**Paper:** Gonzalez (1985), [R16], sobre selección geométrica y objetivos de distancia máxima. Las métricas y umbrales anteriores son definiciones operativas del proyecto, no una certificación DTN del artículo.

### 3.18 Saturación

**1. Qué pregunta responde.** ¿Cuándo ampliar candidatos o representantes deja de aportar mejora relevante?

Hay dos procesos distintos:

- **Saturación del corpus:** crecimiento de \(C_n\) frente a referencia fija.
- **Saturación del pool:** ampliación de candidatos para buscar regiones nuevas.

Con \(C_n\subset C_{n+b}\), sobre la misma referencia de monitorización \(V\):

$$
\Delta_n=R_{95}(C_n;V)-R_{95}(C_{n+b};V).
$$

**2. Supuestos.** Referencia y geometría fijas, checkpoints comparables y criterios preregistrados. Recalcular escalado mezcla efectos.

**3. Features admitidas.** Las de coverage; no exige datos exclusivamente numéricos.

**4. Preprocesamiento.** Registrar curvas globales y por estratos, incertidumbre de muestreo y sensibilidad a inicialización. Reservar evaluación final.

**5. Conclusiones engañosas.** Una meseta puede indicar métrica insensible, generador limitado o presupuesto agotado. Mejora pequeña con cobertura insuficiente no justifica cierre.

**6. Uso propuesto.** Determinar **tamaño del corpus y ampliación del pool**, por separado.

**Paper relacionado:** Loeppky, Sacks y Welch (2009), [R17], sobre tamaño de experimentos para aproximación mediante procesos gaussianos. **No valida esta regla ni permite trasladar automáticamente su regla de tamaño a nuestro corpus.** La parada aquí formulada es una adaptación propia basada en coverage; Gonzalez [R16] aporta fundamento geométrico y Hennig [R18] referencia de estabilidad de clusters.

## 4. Comparación de papeles metodológicos

| Método | Features | Escenarios | Clusters | Selección/cierre |
|---|---|---|---|---|
| Pearson | Diagnóstico lineal | No propuesto entre filas | No | Apoyo, sin descarte automático |
| Spearman | Diagnóstico monótono | No propuesto entre filas | No | Apoyo |
| Covarianza | Variación conjunta | Base de Mahalanobis | Base numérica | Apoyo |
| Varianza | Constantes/variación | No | No | Control del pool |
| Matriz de correlación | Auditoría conjunta | No | Opcional, de features | Apoyo |
| Euclídea | No | Sí | k-means/Ward compatibles | Baseline numérico |
| Manhattan | No | Sí | Métodos compatibles | Alternativa numérica |
| Coseno | No | Perfiles de dirección | Métodos compatibles | Auxiliar inicialmente |
| Gower | No | Principal candidata mixta | Medoids/linkage compatible | Principal candidata |
| Mahalanobis | Usa covarianza | Sensibilidad numérica | Métodos compatibles | Secundaria |
| SVD/rango | Dependencias | Representación | No directamente | No decide \(N\) |
| PCA | Variación/proyección | Exploración | Opcional, validando pérdida | No inicialmente |
| Condición | Control numérico | No | No | Control de cálculos |
| Nearest-neighbour | No | Proximidad local | Apoyo | Redundancia y huecos |
| Clustering | No principalmente | Estructura | Principal | Opcional |
| Silhouette | No | No directamente | Validación auxiliar | No determina \(N\) |
| Coverage | No | Distancia a representantes | Auditoría por grupos | Principal |
| Saturación | No | Mejora al ampliar | Estabilidad auxiliar | Principal con cobertura suficiente |

**Independencia lineal:** como máximo \(p\) vectores de \(\mathbb R^p\) pueden ser linealmente independientes. El corpus puede contener más de \(p\) escenarios. Buscamos **cobertura suficiente y redundancia controlada respecto a un dominio y representación explícitos**.

## 5. Referencias académicas verificadas

Se enlazan DOI, editorial o copias académicas. La verificación bibliográfica no implica acceso libre a todos los textos completos. Se distinguen referencias originales, de cálculo y revisiones; ninguna valida por sí sola nuestras decisiones DTN.

### R1 — Pearson

Pearson, K. (1895). **Note on regression and inheritance in the case of two parents.** *Proceedings of the Royal Society of London*, 58, 240–242.  
DOI: [10.1098/rspl.1895.0041](https://doi.org/10.1098/rspl.1895.0041).  
Uso: referencia histórica para Pearson y matriz de correlación.  
Clave sugerida: pearson1895regression.

### R2 — Spearman

Spearman, C. (1904). **The Proof and Measurement of Association between Two Things.** *The American Journal of Psychology*, 15(1), 72–101.  
DOI: [10.2307/1412159](https://doi.org/10.2307/1412159).  
Uso: asociación por rangos y matriz Spearman.  
Clave sugerida: spearman1904association.

### R3 — Varianza y covarianza

Welford, B. P. (1962). **Note on a Method for Calculating Corrected Sums of Squares and Products.** *Technometrics*, 4(3), 419–420.  
DOI: [10.1080/00401706.1962.10490022](https://doi.org/10.1080/00401706.1962.10490022).  
Uso: cálculo estable de sumas necesarias para varianza y covarianza; no se atribuye la invención de ambas medidas.  
Clave sugerida: welford1962sums.

### R4 — Estimación de covarianza

Ledoit, O., & Wolf, M. (2004). **A well-conditioned estimator for large-dimensional covariance matrices.** *Journal of Multivariate Analysis*, 88(2), 365–411.  
DOI: [10.1016/S0047-259X(03)00096-4](https://doi.org/10.1016/S0047-259X(03)00096-4).  
Uso: regularización y condicionamiento de covarianzas.  
Clave sugerida: ledoitwolf2004covariance.

### R5 — Euclídea, Manhattan y alta dimensión

Aggarwal, C. C., Hinneburg, A., & Keim, D. A. (2001). **On the Surprising Behavior of Distance Metrics in High Dimensional Space.** En *Database Theory — ICDT 2001*, LNCS 1973, 420–434.  
DOI: [10.1007/3-540-44503-X_27](https://doi.org/10.1007/3-540-44503-X_27).  
Uso: comportamiento de normas \(L_p\) y concentración de distancias.  
Clave sugerida: aggarwal2001distance.

### R6 — Coseno y modelo vectorial

Salton, G., Wong, A., & Yang, C. S. (1975). **A vector space model for automatic indexing.** *Communications of the ACM*, 18(11), 613–620.  
DOI: [10.1145/361219.361220](https://doi.org/10.1145/361219.361220).  
Uso: comparación vectorial en recuperación de información, no benchmark DTN.  
Clave sugerida: salton1975vectorspace.

### R7 — Gower

Gower, J. C. (1971). **A General Coefficient of Similarity and Some of Its Properties.** *Biometrics*, 27(4), 857–871.  
DOI: [10.2307/2528823](https://doi.org/10.2307/2528823).  
Uso: similitud para datos de tipos diferentes.  
Clave sugerida: gower1971similarity.

### R8 — Mahalanobis

Mahalanobis, P. C. (1936). **On the generalised distance in statistics.** *Proceedings of the National Institute of Sciences of India*, 2, 49–55.  
Reimpresión: *Sankhya A*, 80(Suppl. 1), 1–7, número de 2018; publicada en línea en 2019.  
DOI de la **reimpresión**, no del original: [10.1007/s13171-019-00164-5](https://doi.org/10.1007/s13171-019-00164-5).  
Clave sugerida para el original: mahalanobis1936distance.

### R9 — SVD y rango

Golub, G. H., & Reinsch, C. (1970). **Singular value decomposition and least squares solutions.** *Numerische Mathematik*, 14, 403–420.  
DOI: [10.1007/BF02163027](https://doi.org/10.1007/BF02163027).  
[Copia académica](https://people.duke.edu/~hpgavin/SystemID/References/Golub%2BReinsch-NM-1970.pdf).  
Clave sugerida: golubreinsch1970svd.

### R10 — PCA: antecedente original

Pearson, K. (1901). **On lines and planes of closest fit to systems of points in space.** *The London, Edinburgh, and Dublin Philosophical Magazine and Journal of Science*, 2(11), 559–572.  
DOI: [10.1080/14786440109462720](https://doi.org/10.1080/14786440109462720).  
Clave sugerida: pearson1901closestfit.

### R11 — PCA: revisión complementaria

Jolliffe, I. T., & Cadima, J. (2016). **Principal component analysis: a review and recent developments.** *Philosophical Transactions of the Royal Society A*, 374, 20150202.  
DOI: [10.1098/rsta.2015.0202](https://doi.org/10.1098/rsta.2015.0202).  
[Texto abierto](https://pmc.ncbi.nlm.nih.gov/articles/PMC4792409/).  
Clave sugerida: jolliffecadima2016pca.

### R12 — Condición numérica

Rice, J. R. (1966). **A Theory of Condition.** *SIAM Journal on Numerical Analysis*, 3(2), 287–310.  
DOI: [10.1137/0703023](https://doi.org/10.1137/0703023).  
Clave sugerida: rice1966condition.

### R13 — Vecino más próximo

Beyer, K., Goldstein, J., Ramakrishnan, R., & Shaft, U. (1999). **When Is “Nearest Neighbor” Meaningful?** En *Database Theory — ICDT'99*, LNCS 1540, 217–235.  
DOI: [10.1007/3-540-49257-7_15](https://doi.org/10.1007/3-540-49257-7_15).  
Clave sugerida: beyer1999nearest.

### R14 — Clustering

Jain, A. K. (2010). **Data clustering: 50 years beyond K-means.** *Pattern Recognition Letters*, 31(8), 651–666.  
DOI: [10.1016/j.patrec.2009.09.011](https://doi.org/10.1016/j.patrec.2009.09.011).  
[Copia en la institución del autor](https://www.cse.msu.edu/~cse802/Papers/DataClustering50yearsBeyondKmeans.pdf).  
Uso: revisión; no artículo original de todos los algoritmos mencionados.  
Clave sugerida: jain2010clustering.

### R15 — Silhouette

Rousseeuw, P. J. (1987). **Silhouettes: A graphical aid to the interpretation and validation of cluster analysis.** *Journal of Computational and Applied Mathematics*, 20, 53–65.  
DOI: [10.1016/0377-0427(87)90125-7](https://doi.org/10.1016/0377-0427(87)90125-7).  
[Copia académica](https://wis.kuleuven.be/stat/robust/papers/publications-1987/rousseeuw-silhouettes-jcam-sciencedirectopenarchiv.pdf).  
Clave sugerida: rousseeuw1987silhouettes.

### R16 — Coverage y selección geométrica

Gonzalez, T. F. (1985). **Clustering to minimize the maximum intercluster distance.** *Theoretical Computer Science*, 38, 293–306.  
DOI: [10.1016/0304-3975(85)90224-5](https://doi.org/10.1016/0304-3975(85)90224-5).  
Uso: selección geométrica y aproximaciones bajo supuestos métricos.  
Clave sugerida: gonzalez1985clustering.

### R17 — Tamaño de experimentos y estudio de parada

Loeppky, J. L., Sacks, J., & Welch, W. J. (2009). **Choosing the Sample Size of a Computer Experiment: A Practical Guide.** *Technometrics*, 51(4), 366–376.  
DOI: [10.1198/TECH.2009.08040](https://doi.org/10.1198/TECH.2009.08040).  
[Versión técnica previa de 2008, NISS](https://www.niss.org/research/technical-reports/choosing-sample-size-computer-experiment-practical-guide-2008).  
Uso: referencia relacionada sobre tamaño y precisión; **no validación de nuestra saturación geométrica**.  
Clave sugerida: loeppky2009samplesize.

### R18 — Estabilidad

Hennig, C. (2007). **Cluster-wise assessment of cluster stability.** *Computational Statistics & Data Analysis*, 52(1), 258–271.  
DOI: [10.1016/j.csda.2006.11.025](https://doi.org/10.1016/j.csda.2006.11.025).  
[Copia del autor](https://www.homepages.ucl.ac.uk/~ucakche/papers/clusta.pdf).  
Uso: reproducibilidad de grupos; estabilidad no equivale a verdad científica.  
Clave sugerida: hennig2007stability.

## 6. Propuesta para este proyecto

### 6.1 Decisión propuesta

**Propongo comparar una disimilitud mixta por familias, seleccionar escenarios reales mediante cobertura incremental y usar correlaciones, SVD y PCA como diagnósticos.**

No congelaría todavía una distancia ni un número. La candidata principal sería una adaptación de Gower; los controles numéricos serían Manhattan y Euclídea sobre el mismo bloque escalado.

La propuesta responde a estas necesidades:

- Hay variables numéricas, categóricas y estructuras de grupos.
- Interesan magnitudes absolutas de comunicación y tráfico.
- El corpus debe contener configuraciones ejecutables.
- Hay que preservar condiciones raras relevantes y evaluar huecos.
- No sabemos si el dominio tendrá clusters naturales.

### 6.2 Representación y ponderación

Organizar inicialmente las features en cuatro bloques revisables:

1. Espacio/mapa.
2. Población/movilidad.
3. Comunicación/recursos.
4. Tráfico.

Los resúmenes de grupos deben preservar relaciones relevantes. Separar distribuciones de velocidades y buffers puede perder qué grupos rápidos tienen buffers grandes. Hay que evaluar esa pérdida antes de considerar equivalentes dos vectores iguales.

Para cada familia \(f\):

$$
d_f(i,\ell)=
\frac{\sum_{j\in f}v_j\delta_j(i,\ell)}
{\sum_{j\in f}v_j},
\qquad
d(i,\ell)=\sum_{f=1}^{F}\alpha_f d_f(i,\ell),
\qquad \sum_f\alpha_f=1.
$$

Los pesos de dimensiones incluidas serán positivos. Peso cero equivale a una exclusión declarada.

**Punto de partida para comparar, no decisión congelada:** igual peso total entre familias. Evita dominio solo por número de columnas. Después se estudiará sensibilidad a pesos y diferencias científicamente críticas.

Para numéricas se estudiarán rangos fijos del dominio y transformaciones justificadas, por ejemplo logaritmos cuando importen razones de carga. No recalcular rangos al añadir candidatos. Marcar valores fuera del dominio; no recortarlos silenciosamente.

Para features no aplicables, preferir bloques por estratos o semántica explícita de aplicabilidad. Omitir componentes por pareja puede cambiar las propiedades de \(d\). Sin demostrar propiedades métricas, no invocar garantías que requieran desigualdad triangular.

**No se adopta todavía una transformación física de mapas o trazas.** Escalar features y transformar espacio/tiempo son operaciones distintas.

### 6.3 Validación de las métricas candidatas

En desarrollo, construir pares con diferencias controladas. Antes de medirlos, documentar qué condiciones esperamos considerar equivalentes, próximas o significativamente distintas.

| Control | Comportamiento que se comprobará |
|---|---|
| Renombrado consistente de grupos y referencias | Misma representación |
| Cambio solo de router, identificador o seed de realización | Misma identidad de escenario |
| Conversión equivalente de unidades | Misma representación tras interpretación |
| Incremento aislado de carga | Diferencia detectable; coseno no debe ocultarla |
| Categoría de movilidad relevante | Diferencia explícita o estrato separado |
| Diferencia importante en una familia | No queda diluida por otra con muchas columnas |
| Grupos distintos con iguales medias | Revisar pérdida de heterogeneidad |
| Tráfico con diferentes relaciones origen/destino | Revisar pérdida por agregación |

Comparar estabilidad de vecinos, distancias por familia y selección bajo Gower adaptada, Manhattan y Euclídea. La comparación numérica no evalúa por sí sola las categorías.

**Regla:** si una métrica contradice controles científicos predefinidos, revisar la representación o descartarla para ese uso. No elegir por producir el gráfico más limpio.

### 6.4 Diagnóstico de features

Calcular varianza, Pearson y Spearman, globalmente y por estratos. Añadir SVD y condición al bloque numérico.

Clasificar cada posible redundancia:

- Identidad algebraica exacta.
- Asociación inducida por el muestreo.
- Asociación empírica que necesita explicación.
- Similitud parcial entre variables con significado diferente.

No eliminar features solo por \(|r|>0.9\) ni por otro umbral aislado. Ante dependencia exacta, decidir qué coordenadas entran en la distancia y mantener las restantes como descriptores auditables.

Usar PCA para explorar. No seleccionar en dos componentes o con un porcentaje de varianza sin comprobar preservación de diferencias relevantes.

### 6.5 Selección incremental y condiciones protegidas

1. Eliminar duplicados de configuración con canonicalización definida. Un hash de bytes detecta archivos iguales; la equivalencia de settings exige resolver semántica.
2. Identificar estratos y extremos obligatorios por razones científicas.
3. Inicializar \(C\) con representantes válidos de esas condiciones.
4. Añadir el candidato más alejado de los representantes actuales:

$$
x^*=\arg\max_{x\in P\setminus C}\min_{c\in C}d(x,c).
$$

5. Registrar mejora de coverage y asignación al representante más cercano.
6. Auditar puntos aislados: pueden ser condiciones útiles o errores.
7. Repetir hasta cumplir los criterios preregistrados o agotar presupuesto.

No exige clusters. Un **medoid** puede representar el centro de un grupo, pero no sustituye necesariamente sus extremos.

La inicialización protegida y las restricciones cambian el problema. **No se atribuirá a esta variante una garantía de aproximación del algoritmo sin restricciones sin demostrarla.**

Cobertura y separación pueden entrar en conflicto. Si dos extremos obligatorios quedan próximos, se documentará por qué se conservan; no se declarará ausencia total de redundancia.

### 6.6 Evaluación y cierre

Separar:

- **Desarrollo:** probar representación, pesos y reglas.
- **Referencia de monitorización:** observar curvas.
- **Evaluación final \(H\):** comprobar el método congelado con candidatos reservados.

El tamaño se determina con la referencia de monitorización \(V\). Después se evalúa una sola vez en \(H\), sin usarlo para elegir el checkpoint, ajustar pesos o tolerancias.

Si un fallo en \(H\) provoca rediseño, pasa a desarrollo y se debe reservar otra evaluación para una afirmación confirmatoria.

Evaluar en el pool y en \(H\):

- \(R_{95}\), \(R_{\max}\) y \(\operatorname{Cov}_{\varepsilon}\).
- Resultados por estrato y condiciones obligatorias.
- Distribución de nearest-neighbour dentro de \(C\).
- Sensibilidad a pesos e inicialización.
- Escenarios concretos peor representados.

Distinguir **cobertura geométrica** y **frecuencia esperada en una población real**. La frecuencia de producción del generador no demuestra frecuencia real.

Regla de cierre propuesta, pendiente de umbrales:

$$
\operatorname{Cov}_{\varepsilon}(C;H)\ge q_{\mathrm{target}},
\qquad R_{\max}(C;H)\le r_{\mathrm{cap}},
$$

junto con condiciones por estrato, estabilidad suficiente y mejora marginal menor que \(\delta\) durante \(m\) checkpoints comparables **en \(V\)**. Los límites absolutos se comprueban primero en \(V\) para decidir el tamaño y después en \(H\) para validar el corpus.

\(\varepsilon,q_{\mathrm{target}},r_{\mathrm{cap}},\delta,m\), tamaño de incrementos y condiciones por estrato permanecen **TBD**. Se fijarán en desarrollo y se preregistrarán con interpretación científica.

**Una meseta sola no autoriza cierre.** Si se agota presupuesto sin cumplir criterios, declarar corpus limitado por presupuesto e informar cobertura conseguida.

### 6.7 Saturación del pool

Antes del freeze, ampliar de forma planificada el pool con candidatos de estratos o mecanismos reservados. Mantener distancia y escalado fijos y comprobar:

- Categorías o combinaciones nuevas.
- Puntos lejos del corpus.
- Regiones sistemáticamente mal cubiertas.
- Heterogeneidades que las features no distinguen.

No se puede demostrar cobertura de todas las redes DTN posibles. La afirmación se limita al dominio, restricciones y referencia definidos.

### 6.8 Papel del clustering

Usarlo como análisis adicional. Con Gower, empezar por medoids o linkage compatible; evaluar silhouette y estabilidad e interpretar los grupos.

**Número de clusters no equivale automáticamente a tamaño del corpus.** Un dominio continuo puede requerir muchos representantes sin clusters naturales; un cluster amplio puede necesitar varios.

### 6.9 Entregables antes del protocolo definitivo

| Entregable | Decisión que deja explícita |
|---|---|
| Schema de escenario | Identidad, tipos, unidades y semántica |
| Feature contract | Qué distingue y qué pierde cada descriptor |
| Distance contract | Transformaciones, escalas, pesos, categorías y ausencias |
| Dominio y restricciones | Universo admisible que se pretende representar |
| Política de referencias | Desarrollo, monitorización, evaluación y pesos |
| Selection contract | Inicialización, extremos, algoritmo y desempates |
| Stopping contract | Tolerancias, checkpoints, estabilidad y presupuesto |
| Sensibilidad | Cambios de vecinos, cobertura y corpus bajo alternativas |
| Freeze | Versiones, hashes, seeds y trazabilidad |

### 6.10 Recomendación final

Empezaría por **Gower adaptada por familias + selección incremental por cobertura + protección de condiciones relevantes + evaluación reservada por estratos**.

Usaría Pearson, Spearman, varianza, covarianza, SVD y condición para auditar features; PCA para explorar; nearest-neighbour para redundancia local; clustering y silhouette para interpretación opcional.

El tamaño final se decidiría por **cobertura, saturación y estabilidad**, con presupuesto explícito. No por independencia lineal, porcentaje de varianza PCA ni número de clusters. La elección definitiva queda pendiente de validar la representación y preregistrar el protocolo.

