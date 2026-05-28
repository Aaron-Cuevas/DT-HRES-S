# DT-HRES-S — Guía de Investigación

Esta guía es el mapa de lectura del proyecto. No es una lista de referencias sueltas: cada sección parte de **una premisa concreta** del proyecto (citada directamente del proposal o derivada de la metodología 4D) y enlaza esa premisa con las fuentes específicas que la responden, con capítulos, secciones, y números de página cuando aplica.

## Cómo usar esta guía

Para cada capítulo encontrarás:

- **Premisa** — la afirmación del proyecto que motiva esta sección.
- **Pregunta abierta** — lo que falta entender para sostener esa premisa.
- **Ruta de lectura** — los recursos en orden, del más accesible al más profundo. Los marcados con ⭐ son los imprescindibles si tienes poco tiempo.
- **Por qué este recurso** — una línea de justificación por cada entrada. Sin esto, una bibliografía es un adorno.
- **Trampas y matices** — errores típicos que estos textos te ayudan a evitar.

Las referencias a libros usan capítulo y sección, no número de página, porque los números de página varían entre ediciones. Cuando un capítulo es largo, indico la sub-sección. Para artículos uso revista, volumen y año, que son inmutables.

---

# Parte 1 — Modelado físico del sistema HRES

## 1.1 — Sistema fotovoltaico

> **Premisa del proposal**: *"modeling based upon the first principles of energy conservation and life-cycle assessment [...] while matching the demand requirements with production power"*.

El modelo usable más simple es `P_pv = P_nom · (G/1000) · [1 + γ(T−25)] · η`. Esa fórmula esconde decisiones sobre la temperatura de celda (no es la del aire), las componentes de irradiancia (GHI vs. la que realmente cae sobre el plano del panel), la inclinación, el ensuciamiento y la degradación. Cada capa de esas se trata en una referencia distinta.

### Pregunta abierta
¿Cómo paso de la irradiancia global horizontal que me entrega NASA POWER a la potencia eléctrica AC que el inversor entrega al bus, con el detalle suficiente para que el dimensionamiento sea creíble pero sin entrar en el diodo de cuatro parámetros?

### Ruta de lectura

**⭐ 1. Masters, G.M. — *Renewable and Efficient Electric Power Systems*, 2.ª ed., Wiley, 2013.**
- **Capítulo 5: The Solar Resource** — geometría sol-Tierra, GHI/DNI/DHI, ángulo de incidencia, conversión a plano inclinado. Es el punto de partida.
- **Capítulo 8: Photovoltaic Systems** — derivación del modelo simple de eficiencia (§8.5), del modelo de circuito equivalente (§8.4), y dimensionamiento de stand-alone (§8.10).
- *Por qué este primero*: rigor de licenciatura, ecuaciones implementables tal cual. Es exactamente el nivel del `src/pv_model.py` actual.

**2. Duffie, J.A. & Beckman, W.A. — *Solar Engineering of Thermal Processes*, 5.ª ed., Wiley, 2020.**
- **Capítulos 1–2**: radiación extraterrestre, atenuación atmosférica, geometría detallada.
- **Capítulo 23: Photovoltaic Systems** — tratamiento formal del modelo de un diodo y cinco parámetros.
- *Por qué después de Masters*: este es la referencia canónica, pero su rigor solo aprecia tras tener la intuición de Masters. Si saltas directo aquí te ahogas.

**3. King, D.L., Boyson, W.E., Kratochvil, J.A. — *Photovoltaic Array Performance Model* (SAND2004-3535, Sandia National Laboratories, 2004).**
- PDF abierto en `energy.sandia.gov`. Lo implementan tal cual en `pvlib`.
- *Cuándo leerlo*: cuando γ·(T−25) ya no sea suficiente y necesites incorporar respuesta espectral y dependencia con el ángulo de incidencia.

**4. Documentación de `pvlib-python`** — https://pvlib-python.readthedocs.io
- `pvlib.irradiance`: para convertir GHI medida a plano-de-arreglo según la inclinación real del panel (modelos Hay-Davies, Pérez).
- `pvlib.temperature.sapm_cell`: para estimar temperatura de celda a partir de temperatura ambiente, viento e irradiancia. Detalle: la documentación cita el paper original junto a cada función.

### Trampas a investigar
- La diferencia entre **temperatura ambiente** (lo que reporta NASA POWER) y **temperatura de celda** (lo que entra en la ecuación de PV) suele ser de 15–25 °C en operación. Ignorarla sobreestima la potencia entre 5 y 10%.
- Los datos de NASA POWER son **reanálisis** (MERRA-2 + CERES), no medición directa. Para Ixil hay que validar contra una estación cercana de UADY o Conagua antes de confiar.

### Recursos en video
- **NREL — playlist *Solar Resource Assessment*** (canal de NREL en YouTube). Videos de ~15 min cada uno; el de *Components of Solar Radiation* da la intuición de GHI/DNI/DHI mejor que cualquier libro.
- **MIT OpenCourseWare — 2.627 *Fundamentals of Photovoltaics* (Buonassisi)**, lecciones 3–6 para el modelado eléctrico. Gratis.

---

## 1.2 — Sistema eólico

> **Premisa**: *"hybrid renewable energy systems that combine solar photovoltaic, wind, and storage options"*. Si el viento entra al mix con la misma legitimidad que el PV, el modelo debe tener fidelidad equivalente — con su reto propio: la potencia escala con el **cubo** de la velocidad, no linealmente, lo que amplifica errores.

### Pregunta abierta
¿Qué curva de potencia uso? ¿Cómo extrapolo la velocidad del anemómetro (típicamente a 10 m) a la altura del rotor (típicamente 20–50 m para los aerogeneradores pequeños que aplicarían en Ixil)?

### Ruta de lectura

**⭐ 1. Masters, G.M. — *Renewable and Efficient Electric Power Systems*, capítulo 6.**
- §6.3: distribución de Weibull para caracterizar el viento.
- §6.4: límite de Betz (lo dice corto y sin mística).
- §6.7–6.8: curva de potencia y factor de capacidad. Cálculos resueltos.

**2. Manwell, J.F., McGowan, J.G., Rogers, A.L. — *Wind Energy Explained: Theory, Design and Application*, 2.ª ed., Wiley, 2009.**
- **Capítulo 2**: el recurso eólico, Weibull en detalle, dependencia de la rugosidad.
- **Capítulo 3**: aerodinámica — para entender de dónde sale la curva de potencia.
- **Capítulo 6**: turbinas pequeñas (las relevantes para una comunidad).

**3. Documentación de `windpowerlib`** — https://windpowerlib.readthedocs.io
- Curvas de potencia de turbinas comerciales por modelo, corrección por densidad del aire, conversión por altura.

### Trampas a investigar
- La **ley de potencia de Hellmann** `v(h) = v(h_ref) · (h/h_ref)^α` depende del coeficiente α que cambia con la rugosidad del terreno: ~0.14 para costa abierta, 0.20–0.30 para terreno con vegetación. Ixil está en Yucatán costero-tropical: el valor correcto no es 1/7 (el default académico) y vale la pena estudiarlo. Ver:
  - **Touma, J.S. — "Dependence of the Wind Profile Power Law on Stability for Various Locations"**, *Journal of the Air Pollution Control Association* 27 (1977): 863–866. Da una tabla de α por tipo de terreno y estabilidad atmosférica.
- La **densidad del aire** corrige la potencia: P ∝ ρ. A 30 °C la densidad es ~3% menor que a 20 °C, lo que reduce la generación. En clima Aw esto es relevante.

### Video
- **National Renewable Energy Laboratory — webinar *Wind Resource Assessment Basics*** (YouTube, NREL channel). 45 min, cubre desde Weibull hasta la incertidumbre del recurso.

---

## 1.3 — Banco de baterías

> **Premisa**: *"hybrid renewable energy systems that combine [...] storage options"*. La batería es el componente que media entre la intermitencia renovable y la demanda real. Modelarla mal lleva a sub/sobredimensionar todo el sistema.

### Pregunta abierta
¿Cuál es el modelo mínimo viable de SoC (State of Charge)? ¿Cuándo es indispensable modelar envejecimiento y degradación cíclica?

### Ruta de lectura

**⭐ 1. Masters, G.M. — *Renewable and Efficient Electric Power Systems*, §9.5 *Batteries* dentro del capítulo de sistemas stand-alone.**
- Eficiencia round-trip, profundidad de descarga, ciclos de vida vs. DoD. Modelo de SoC con balance de energía.

**2. Linden, D. & Reddy, T.B. (eds.) — *Linden's Handbook of Batteries*, 4.ª ed., McGraw-Hill, 2011.**
- Para químicas: capítulo de **plomo-ácido** (típica en sistemas comunitarios por costo) y capítulo de **iones de litio** (LiFePO4 es el estándar emergente).
- Es referencia de consulta, no se lee de corrido.

**3. Plett, G.L. — *Battery Management Systems, Volume I: Battery Modeling*, Artech House, 2015.**
- Para cuando el modelo de SoC simple no alcanza y necesitas modelo de circuito equivalente con resistencia interna y polarización.
- §2.1–2.4 son el mínimo útil.

### Trampas a investigar
- El SoC no es lineal en el voltaje. Para plomo-ácido la "meseta" de voltaje cubre el 80% del SoC; usar voltaje como proxy directo del SoC introduce errores grandes cerca de los extremos.
- La eficiencia round-trip no es constante: cae fuera del rango óptimo de C-rate (la velocidad de carga/descarga relativa a la capacidad).
- El envejecimiento depende más del **número de ciclos profundos** que del tiempo. Para una comunidad con perfil de uso predecible, este factor define el reemplazo a 5–7 años.

---

## 1.4 — Balance del sistema y lógica de despacho

> **Premisa**: *"following a specific between-source operational algorithm"*. El despacho (qué fuente abastece la carga en cada hora, cuándo se carga la batería, cuándo se vierte excedente) es la columna vertebral de la simulación y a veces la decisión que más impacta los resultados.

### Pregunta abierta
¿Qué algoritmo de despacho usar como baseline antes de cualquier optimización? ¿Cómo lidiar con el exceso de generación (curtailment) cuando la batería está llena?

### Ruta de lectura

**⭐ 1. Lambert, T., Gilman, P., Lilienthal, P. — "Micropower System Modeling with HOMER" (capítulo 15 de *Integration of Alternative Sources of Energy*, Farret & Simões eds., Wiley, 2006).**
- PDF gratuito en `homerenergy.com/pdf`. Es la metodología original de HOMER, que es exactamente el software que el proyecto busca democratizar. Léelo: define el problema que estás resolviendo.

**2. Patel, M.R. — *Wind and Solar Power Systems: Design, Analysis, and Operation*, 2.ª ed., CRC Press, 2005.**
- **Capítulo 13**: dimensionamiento de stand-alone, balance horario.
- **Capítulo 14**: estrategias de despacho.

**3. Erbs, D.G., Klein, S.A., Beckman, W.A. — "Estimation of Degree-Days and Ambient Temperature Bin Information from Monthly-Average Temperatures"**, *ASHRAE Journal* 25, 1983.
- Antiguo pero la base de cómo construir perfiles horarios desde estadísticos mensuales — útil cuando no tienes datos de Ixil pero sí promedios.

---

# Parte 2 — Datos meteorológicos y validación en múltiples climas

## 2.1 — Por qué validar en varios climas (no solo en uno)

> **Premisa derivada de la metodología**: *"el gemelo digital se entrena con datos de varios climas antes de desplegarse en Ixil"*. Esta es una decisión deliberada para probar la **generalización** del modelo. Un surrogate entrenado solo en un sitio y desplegado a otro es una causa típica de fallas en producción.

### Pregunta abierta
¿Cómo se eligen "tres climas" no de forma arbitraria sino de forma que cubran el envelope operativo de Ixil con margen?

### Ruta de lectura

**⭐ 1. Peel, M.C., Finlayson, B.L., McMahon, T.A. — "Updated world map of the Köppen-Geiger climate classification", *Hydrology and Earth System Sciences* 11 (2007): 1633–1644.**
- Acceso abierto en `hess.copernicus.org/articles/11/1633/2007`.
- Define formalmente las clases climáticas modernas. Ixil cae en **Aw** (sabana tropical). El conjunto natural de validación: un **BSh** o **BWh** (semiárido/desértico caliente), un **Cfb** (oceánico templado), y **Aw**. Eso cubre el rango térmico y de humedad que el modelo verá. La elección no debe ser "tres ciudades de México" sino tres clases Köppen contrastantes.

**2. Beck, H.E. et al. — "Present and future Köppen-Geiger climate classification maps at 1-km resolution", *Scientific Data* 5 (2018): 180214.**
- Mapas en alta resolución, abiertos. Si necesitas saber exactamente qué clase Köppen tiene Ixil al detalle, este es el dato.

### Trampa a investigar
Validar en climas similares no prueba generalización. Si entrenas en tres sitios con Aw todos, tu test set no es independiente del training set en sentido estadístico. La separación geográfica no garantiza separación climática.

---

## 2.2 — Datos del recurso solar y eólico

> **Premisa**: el sistema necesita datos meteorológicos horarios de calidad suficiente para entrenar el modelo. Para Ixil la opción inmediata es NASA POWER; la opción mediano-plazo es instrumentación propia.

### Pregunta abierta
¿Qué tan confiable es NASA POWER para Ixil? ¿Cómo se construye un Año Meteorológico Típico (TMY) y por qué no es un año real?

### Ruta de lectura

**⭐ 1. Wilcox, S. & Marion, W. — *Users Manual for TMY3 Data Sets*, NREL/TP-581-43156, 2008.**
- PDF abierto en `nrel.gov/docs`. Explica cómo se construye un TMY a partir de múltiples años: se elige el mes "más típico" de cada uno de los 12 meses de varios años y se concatenan. Por eso un TMY no representa ningún año real, sino el comportamiento estadísticamente representativo.

**2. Documentación de NASA POWER** — https://power.larc.nasa.gov/docs/methodology/
- La sección de Metodología es obligada. NASA POWER es **reanálisis** (combina MERRA-2 con CERES satelital), no medición directa. Eso tiene implicaciones para incertidumbre.

**3. Sengupta, M. et al. — *Best Practices Handbook for the Collection and Use of Solar Resource Data for Solar Energy Applications*, 3.ª ed., NREL/TP-5D00-77635, 2021.**
- PDF abierto. La biblia operativa del recurso solar: cómo medirlo, cómo validarlo, cómo combinar satélite con tierra.

### Trampas a investigar
- NASA POWER reporta promedios espaciales sobre celdas de ~0.5° × 0.5° (~55 × 55 km). Para terreno heterogéneo, un sitio dentro de la celda puede diferir significativamente del promedio.
- La irradiancia tiene errores sistemáticos en regiones nubladas (la cobertura nubosa de los reanálisis es notoriamente débil en el trópico húmedo).

---

## 2.3 — Incertidumbre en mediciones meteorológicas

> **Premisa de la metodología 4D — dimensión 3D (Mente)**: *"every reading carries a margin of error that propagates to the result"*.

### Ruta de lectura

**⭐ 1. WMO — *Guide to Meteorological Instruments and Methods of Observation*, WMO-No. 8, 2018.**
- Abierto en `library.wmo.int`. Capítulo 7: medición de radiación solar (incertidumbres típicas: ±2% para piranómetro de clase A, ±5% para clase B, ±5–10% para sensores fotovoltaicos baratos). Capítulo 5: medición de viento.

**2. Saltelli, A. et al. — *Global Sensitivity Analysis: The Primer*, Wiley, 2008.**
- Si quieres ir más allá de la propagación lineal de errores (expansión de Taylor), capítulos 1 y 5 son el punto de entrada al análisis de Sobol.

---

# Parte 3 — Dimensionamiento HRES de primeros principios

## 3.1 — Métricas objetivo: LPSP, LCOE, fracción renovable

> **Premisa del proposal**: *"specifically report the indicators like levelized cost of energy, while matching the demand requirements with production power"*.

### Ruta de lectura

**⭐ 1. Borowy, B.S. & Salameh, Z.M. — "Methodology for Optimally Sizing the Combination of a Battery Bank and PV Array in a Wind/PV Hybrid System", *IEEE Transactions on Energy Conversion* 11 (1996): 367–375.**
- El paper que introduce el concepto de Loss of Power Supply Probability (LPSP) en el contexto HRES. Cobertura = 1 − LPSP. Lectura obligada antes de implementar la métrica.

**2. Short, W., Packey, D.J., Holt, T. — *A Manual for the Economic Evaluation of Energy Efficiency and Renewable Energy Technologies*, NREL/TP-462-5173, 1995.**
- Abierto. La fórmula del LCOE con su derivación. Define qué entra en CapEx, OpEx, tasa de descuento, factor de recuperación de capital.

**3. International Energy Agency — *Projected Costs of Generating Electricity*, edición más reciente.**
- Para valores de referencia de CapEx/OpEx por tecnología, validar que los números que metes a tu LCOE están dentro del rango plausible.

### Trampa a investigar
La **tasa de descuento** es la variable más sensible del LCOE — más que CapEx. Para una comunidad indígena con financiamiento de subsidio, la tasa apropiada no es la comercial (8–12%) sino la social (3–5%). Esto cambia los resultados drásticamente.

---

# Parte 4 — Aprendizaje automático: el modelo sustituto

## 4.1 — Por qué un regresor sustituto

> **Premisa del proposal**: *"A physics-constrained digital twin model of the HRES developed through the Artificial Intelligence (AI) method can help by offering computational, economical, and cost-effective technology as an alternative to the iterative first principles-based energy and economic modeling."*

El procedimiento iterativo de HOMER/PVsyst evalúa miles de configuraciones × 8760 horas × varios años de clima. Un regresor entrenado lo reemplaza con una inferencia en milisegundos. Esto es el patrón **surrogate model**, bien establecido en ingeniería.

### Pregunta abierta
¿Por qué Random Forest y no otros? ¿Cuándo dominan SVM o redes neuronales?

### Ruta de lectura

**⭐ 1. James, G., Witten, D., Hastie, T., Tibshirani, R. — *An Introduction to Statistical Learning, with Applications in Python*, Springer, 2023 (ISLP).**
- PDF gratuito y legal en `statlearning.com`.
- **Capítulo 8: Tree-Based Methods** — árboles de decisión (§8.1), bagging y random forests (§8.2.1–8.2.2), boosting (§8.2.3). Los labs en Python al final del capítulo usan exactamente la sintaxis de scikit-learn que tu proyecto va a escribir.
- *Por qué este primero*: nivel y velocidad pedagógicos correctos. Si lees ESL antes de ISLR, sufres innecesariamente.

**2. Hastie, T., Tibshirani, R., Friedman, J. — *The Elements of Statistical Learning*, 2.ª ed., Springer, 2009 (ESL).**
- PDF gratuito y legal en `hastie.su.domains/ElemStatLearn`.
- **Capítulo 9: Additive Models, Trees, and Related Methods** — derivación formal.
- **Capítulo 15: Random Forests** — completo. Ver especialmente §15.3 (detalles del algoritmo) y §15.4 (cuándo random forests sobreajusta y por qué generalmente no).

**3. Breiman, L. — "Random Forests", *Machine Learning* 45 (2001): 5–32.**
- El artículo original. Pocas páginas, claras. Léelo después de ISLR para ver dónde están las decisiones de diseño del algoritmo.

### Recursos en video
- **StatQuest with Josh Starmer** (YouTube). Las series *Decision Trees, Clearly Explained* y *Random Forests, Part 1 & 2* son la mejor explicación visual disponible, ~15 min cada una. Empezar aquí si nunca has visto un árbol entrenado.
- **Stanford CS229 (Andrew Ng) — clases 9 y 10** en YouTube, sobre árboles y métodos de ensemble.

---

## 4.2 — SVM y redes neuronales como alternativas

> **Premisa del proposal**: *"The algorithms from supervised ML can be tested, ranging from Decision Tree, Random Forest, Support Vector Machine, to advanced algorithms based upon Neural Networks."*

### Ruta de lectura

**1. Bishop, C.M. — *Pattern Recognition and Machine Learning*, Springer, 2006.**
- **Capítulo 7**: máquinas de kernel dispersas (SVM). El tratamiento más claro del kernel trick a nivel de licenciatura avanzada.
- PDF gratuito desde 2024 en `microsoft.com/en-us/research`.

**2. Goodfellow, I., Bengio, Y., Courville, A. — *Deep Learning*, MIT Press, 2016.**
- Libro completo gratuito en `deeplearningbook.org`.
- Para el contexto del proyecto, leer solo:
  - **Capítulo 6**: redes feedforward profundas (lo mínimo para implementar una NN básica).
  - **Capítulo 7**: regularización.
  - **Capítulo 11**: metodología práctica (cuándo usar NN, cuándo no).
- Si el modelo final es Random Forest, no necesitas más NN que esto.

### Recursos en video
- **3Blue1Brown — serie *Neural Networks*** (4 videos en YouTube). Intuición visual incomparable. Empezar aquí antes que cualquier libro.
- **Andrej Karpathy — *Neural Networks: Zero to Hero*** (YouTube). Si después de StatQuest y 3Blue1Brown quieres implementar una NN desde cero en Python.

---

## 4.3 — División train / validation / test y validación cruzada

> **Premisa del proposal**: *"The data is divided into three sets, including the training, validation, and testing, with different percentages."*

### Ruta de lectura

**⭐ 1. ISLR — Capítulo 5: Resampling Methods.**
- §5.1: validation set approach.
- §5.2: leave-one-out cross-validation.
- §5.3: k-fold cross-validation.
- Es lo mínimo necesario y suficiente.

**2. ESL — Capítulo 7: Model Assessment and Selection.**
- §7.10: cross-validation con discusión del *one-standard-error rule* y del sesgo optimista del CV cuando se hace mal.

### Trampa a investigar y resolver explícitamente
Para validación geográfica (que es lo que se necesita en HRES), un k-fold aleatorio **no funciona**: filtras información entre folds porque sitios cercanos correlacionan. El esquema correcto es **leave-one-region-out** o **leave-one-city-out**. Esto está en:

- **Roberts, D.R. et al. — "Cross-validation strategies for data with temporal, spatial, hierarchical, or phylogenetic structure"**, *Ecography* 40 (2017): 913–929. Abierto. Explica por qué CV ingenua sobre-estima el desempeño en datos con estructura espacial.

---

## 4.4 — Métricas de desempeño: R², MAPE, MSE, CV-RMSE

> **Premisa del proposal**: *"measuring their statistical performance like Coefficient of Determination, Mean Absolute Percentage Error, and Mean Squared Error"*.

### Ruta de lectura

**⭐ 1. ASHRAE — *Guideline 14-2014: Measurement of Energy, Demand, and Water Savings*.**
- Define formalmente CV-RMSE (Coefficient of Variation of the Root Mean Squared Error) y NMBE (Normalized Mean Bias Error) con los umbrales de aceptación para modelos energéticos: CV-RMSE ≤ 30% horario, ≤ 15% mensual. Es el estándar de la industria HVAC/energía aplicado al modelado energético. Lo usarías como criterio de aceptación del surrogate.

**2. Hyndman, R.J. & Koehler, A.B. — "Another look at measures of forecast accuracy", *International Journal of Forecasting* 22 (2006): 679–688.**
- Explica por qué MAPE falla cuando los valores reales se acercan a cero, y propone MASE como alternativa. Importante si parte de la salida del HRES son potencias que rozan cero (noches sin batería, por ejemplo).

**3. Documentación de `sklearn.metrics`** — https://scikit-learn.org/stable/modules/model_evaluation.html
- Implementaciones exactas. Leer la sección sobre *regression metrics* para conocer las opciones (`r2_score`, `mean_squared_error`, `mean_absolute_percentage_error`, etc.).

---

# Parte 5 — Implementación en Python

## 5.1 — Stack numérico y de datos

> **Premisa del proposal**: *"can be programmed through open access libraries of Python in the Google Colab environment."*

### Ruta de lectura

**⭐ 1. VanderPlas, J. — *Python Data Science Handbook*, O'Reilly, 2.ª ed., 2022.**
- Gratuito en `jakevdp.github.io/PythonDataScienceHandbook`.
- Capítulos 2 (NumPy), 3 (pandas), 5 (scikit-learn). En conjunto, la base del 90% del código del proyecto.

**2. McKinney, W. — *Python for Data Analysis*, O'Reilly, 3.ª ed., 2022.**
- McKinney es el autor de pandas. Capítulos 5–8 son la referencia para manipulación de series temporales horarias.

---

## 5.2 — Librerías de dominio: `pvlib` y `windpowerlib`

Ya enlazadas en §1.1 y §1.2. Las repito aquí porque son la columna del código de simulación:

- `pvlib-python` — modelado fotovoltaico (irradiancia, temperatura de celda, modelo de diodo).
- `windpowerlib` — curvas de potencia, corrección por altura y densidad.

**Caso resuelto en pvlib** — *PV Performance Modeling Collaborative*, `pvpmc.sandia.gov`. La PVPMC es una colaboración Sandia/NREL donde están los benchmarks y los ejemplos de validación que el proyecto puede replicar para sus tres climas.

---

## 5.3 — scikit-learn para el surrogate

**⭐ 1. Documentación oficial de scikit-learn — *User Guide*, secciones:**
- §1.10 *Decision Trees*
- §1.11 *Ensemble methods* (en particular §1.11.2 Random Forests, §1.11.4 Gradient Boosting)
- §1.4 *Support Vector Machines*
- §3.1 *Cross-validation*
- §3.3 *Metrics and scoring*
- §3.4 *Validation curves*
- Como referencia esto es de mayor valor que muchos libros, porque está siempre actualizada y cada parámetro de `RandomForestRegressor` está documentado con su significado matemático.

**2. Géron, A. — *Hands-On Machine Learning with Scikit-Learn, Keras, and TensorFlow*, O'Reilly, 3.ª ed., 2022.**
- Capítulos 2–7 son la mejor implementación práctica disponible. Cada paso de un proyecto de ML real con código en GitHub. Si el equipo ya leyó ISLR, este es el siguiente paso natural.

---

## 5.4 — Flujos reproducibles en Colab y control de versiones

> **Premisa del proposal**: *"in the Google Colab environment"* + el repo del proyecto está en GitHub.

### Ruta de lectura

**⭐ 1. The Turing Way — *A handbook for reproducible, ethical and collaborative data science*.**
- Abierto en `the-turing-way.netlify.app`. Capítulos sobre *reproducibility* y *project design*. Estándar emergente.

**2. Chacon, S. & Straub, B. — *Pro Git*, 2.ª ed., Apress, 2014.**
- Gratuito en `git-scm.com/book`. Solo capítulos 1–3 son necesarios para el flujo del equipo (clone, branch, commit, pull request).

---

# Parte 6 — Teoría del gemelo digital

## 6.1 — Qué es un gemelo digital (y qué no)

> **Premisa**: el proyecto se llama Digital Twin-based HRES-Solution. La palabra "digital twin" tiene mucha definición vaga en la literatura. Conviene anclar el concepto.

### Ruta de lectura

**⭐ 1. Grieves, M. & Vickers, J. — "Digital Twin: Mitigating Unpredictable, Undesirable Emergent Behavior in Complex Systems"**, capítulo en *Transdisciplinary Perspectives on Complex Systems*, Springer, 2017.
- La formulación original del concepto. Define las tres componentes (sistema físico, gemelo virtual, datos que los conectan) y la progresión Modelo → Sombra → Gemelo.

**2. Tao, F., Zhang, M., Nee, A.Y.C. — *Digital Twin Driven Smart Manufacturing*, Academic Press, 2019.**
- Capítulos 1–3. Marco moderno aplicado a sistemas físicos, con menos retórica que Grieves.

**3. Kapteyn, M.G., Pretorius, J.V.R., Willcox, K.E. — "A probabilistic graphical model foundation for enabling predictive digital twins at scale"**, *Nature Computational Science* 1 (2021): 337–347.
- El gemelo digital con incertidumbre, formalizado como red bayesiana dinámica. Es el horizonte HRES 6–7 de tu metodología.

---

## 6.2 — Machine learning informado por la física

> **Premisa del proposal**: *"A **physics-constrained** digital twin model"*. La palabra "physics-constrained" es importante: no es un ML ciego sobre datos.

### Ruta de lectura

**⭐ 1. Karniadakis, G.E. et al. — "Physics-informed machine learning", *Nature Reviews Physics* 3 (2021): 422–440.**
- El paper de referencia. Define PINNs (Physics-Informed Neural Networks), physics-guided ML, y los tres niveles de inclusión de la física: como dato de entrenamiento, como prior, como término de pérdida.

**2. Raissi, M., Perdikaris, P., Karniadakis, G.E. — "Physics-informed neural networks"**, *Journal of Computational Physics* 378 (2019): 686–707.
- El paper que detonó la línea PINN. Más técnico que el Nature Reviews; léelo después.

### Conexión con el proyecto
Para HRES, "physics-constrained" puede significar simplemente que las salidas del surrogate se chequean contra los principios de conservación (la potencia entregada nunca puede exceder la generada menos las pérdidas; el SoC siempre en [0,1]; etc.) y se rechaza/penaliza si los viola. Esa validación de consistencia física es responsabilidad de Miguel Garduño en el equipo.

---

# Parte 7 — Metodología participativa y contexto indígena

> **Premisa del proposal**: *"a decentralized bottom-up participatory approach"* contra el enfoque tradicional *"top-down energy planning by technocratic centralized decision making"*.

Esta parte no es opcional. El proposal lo pone en igual jerarquía con la parte técnica, y un sistema técnicamente correcto pero socialmente desarraigado fracasará en Ixil — eso lo dice el propio proposal cuando habla de fallas previas de aceptación.

### Ruta de lectura

**⭐ 1. Chambers, R. — *Whose Reality Counts? Putting the First Last*, Intermediate Technology Publications, 1997.**
- El texto fundacional del enfoque participativo en desarrollo. Capítulos 7–8 sobre PRA (Participatory Rural Appraisal).

**2. Freire, P. — *Pedagogía del oprimido*, Siglo XXI, 1970.**
- Para el componente educativo del IESO. La idea de que la educación liberadora se construye **con** la comunidad, no **para** ella, atraviesa todo el proposal aunque no lo cite.

**3. IRENA — *Renewable Energy and Indigenous Peoples*, IRENA, 2021.**
- Abierto. Casos comparativos globales de despliegue renovable con comunidades indígenas. Da expectativas calibradas sobre lo que funciona y lo que no.

**4. Boletines de la CDI / INPI (Instituto Nacional de los Pueblos Indígenas)**, México.
- Para contexto específico de pueblos mayas peninsulares (que es donde se ubica Ixil).

### Recurso de video
- **TEDxMonterrey — *Innovación social y comunidades*** (varias charlas en YouTube). Útiles para entender el lenguaje y los marcos que se usan en México alrededor de este tema, distintos al lenguaje académico anglosajón.

---

# Apéndice — Orden sugerido de lectura para el equipo

No todos leen todo. La asignación natural según la tabla de responsabilidades del proyecto (`docs/4D_methodology/07_team_responsibilities.md`):

**Víctor Cardeña (1D Concept · physical models)** — Parte 1 completa.

**Daniel Leiva (1D Concept · documentation)** — Lectura transversal: capítulos introductorios (⭐ marcados) de Partes 1, 2, 4 y 6. El objetivo es que pueda explicar cualquier parte del sistema a alguien externo.

**Aaron Cuevas (2D Body · integration)** — Partes 3 y 5 completas + capítulo 6.1.

**Carlos Rodríguez Tenorio (2D Body · interface & metrics)** — Parte 3 (especialmente 3.1) + capítulos 4.4 y 5.3.

**Samuel Canul (3D Mind · data sources)** — Parte 2 completa, especialmente 2.2.

**José Llashag (3D Mind · sensors & deployment)** — Parte 2 + capítulo 2.3 (incertidumbre).

**Arturo Cruz (4D Spirit · integration & reproducibility)** — Capítulos 5.4, 6.1, 6.2. Adicionalmente *The Turing Way* completo.

**Emilio Urbina (4D Spirit · baseline DT/RF)** — Parte 4 completa, especialmente capítulos 4.1 y 4.2.

**Félix Valadez (4D Spirit · simulation analytics)** — Capítulo 1.4 (despacho), Parte 3, capítulo 4.4 (métricas).

**Regina Muñoz (HRES 6–7 · validation & drift)** — Parte 4 completa, especialmente capítulos 4.3 (CV) y 4.4 (métricas). Adicionalmente capítulo 6.2 (physics-constrained ML).

**Miguel Garduño (HRES 6–7 · benchmarking)** — Capítulos 4.4 y 6.2. Roberts et al. 2017 sobre CV con estructura espacial es prioritario.

**Todo el equipo, antes de cualquier visita a la comunidad** — Parte 7 (mínimo Chambers cap. 7–8 y el reporte IRENA).

---

# Cómo contribuir a esta guía

Si encuentras un recurso que mereciera estar aquí, abre un Pull Request agregándolo bajo la sección correspondiente con el formato que sigue:

```
**N. Autor(es) — *Título exacto*, Editorial, Año.**
- Sección o capítulos relevantes con un breve "qué encontrarás ahí".
- Por qué este recurso responde la premisa de la sección.
- Link abierto si existe.
```

Las referencias sin "por qué" se rechazan: la coherencia de la guía depende de que cada entrada explique por qué está aquí.

---

*Esta guía se actualiza con cada etapa HRES alcanzada. Está pensada como mapa de lectura, no como bibliografía exhaustiva. La selección es intencional: pocas referencias, bien elegidas, leídas en orden.*
