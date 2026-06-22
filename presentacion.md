# Presentación — Proyecto INF405
## Comparación de técnicas tiempo-frecuencia (y topológicas) para la caracterización del ritmo alfa en EEG (ojos abiertos vs. cerrados)

**Integrantes:** Matías Peñaloza · Benjamín Ulloa · Alfredo Iturra · Juan Suárez
**Curso:** Procesamiento de Señales e Imágenes (INF405)

---

> **Formato:** 14 diapositivas (1 portada + 12 de contenido + 1 cierre). ~18 min, ~1–1.5 min por diapo.
> Cada diapo trae: **🎯 Contenido** (qué se muestra) · **🗣️ Contexto** (de qué se habla) · **⭐ Puntos clave** (lo que sí o sí hay que decir).
> Las diapos marcadas *(opcional)* se pueden saltar si falta tiempo (bajaría a ~11 diapos).

---

### Diapo 1 — Portada
- **🎯 Contenido:** Título, integrantes, curso, fecha. Imagen de fondo (EEG / electrodos).
- **🗣️ Contexto:** Primera impresión; el tema debe quedar claro al instante.
- **⭐ Puntos clave:** Frase gancho: *"¿Qué técnica representa mejor el ritmo alfa para clasificar si una persona tiene los ojos abiertos o cerrados?"*

---

### Diapo 2 — Problema, fenómeno y pregunta
- **🎯 Contenido:**
  1. El EEG es difícil: señales de baja amplitud, **no estacionarias** y con muchos artefactos.
  2. El **ritmo alfa** (~8–13 Hz) **aumenta con ojos cerrados** y se bloquea al abrirlos → da la etiqueta.
  3. **Pregunta:** ¿qué representación describe mejor el alfa y permite clasificar mejor el estado ocular?
- **🗣️ Contexto:** Junta motivación + marco fisiológico + pregunta de investigación.
- **⭐ Puntos clave:**
  - "No estacionaria" = razón de ser del análisis tiempo-frecuencia.
  - **Clasificación binaria**: clase 0 = ojos abiertos, clase 1 = ojos cerrados.

---

### Diapo 3 — Hipótesis y objetivos
- **🎯 Contenido:**
  - **Hipótesis:** las **wavelets** (multirresolución) representarían mejor la dinámica no estacionaria del alfa que la STFT.
  - **4 objetivos** = 4 fases: acondicionar → transformar → extraer features → clasificar y comparar.
- **🗣️ Contexto:** La hipótesis es la vara para medir resultados; los objetivos anticipan el pipeline.
- **⭐ Puntos clave:** Dejar explícita la apuesta ("creemos que ganan las wavelets") para contrastarla con honestidad al final.

---

### Diapo 4 — Datos y pipeline
- **🎯 Contenido:**
  - **Dataset UCI EEG Eye State** (Roesler 2013): 14 canales, 128 Hz, sesión de 117 s, etiqueta validada por video; tras preprocesar y ventanear (75 % de solapamiento) quedan **231 ventanas**.
  - **Diagrama del pipeline:** `01 Acondicionamiento → 01b Limpieza ICA → 02 Transformación → 03 Features → 04/05 EDA → 06 Clasificación → 07 Evaluación robusta`.
  - Dos familias de técnicas: **tiempo-frecuencia** (notebooks base) y **topológicas** (rama "b", extensión espacial).
- **🗣️ Contexto:** El "mapa" del proyecto. Si recuerdan una sola diapo, que sea esta.
- **⭐ Puntos clave:**
  - Datos públicos y pre-validados → reproducibilidad. **Un solo sujeto/sesión, ~117 s** → ojo con la generalización (limitación clave después).
  - Stack: SciPy, PyWavelets, scikit-learn.

---

### Diapo 5 — Fase 1: Acondicionamiento de la señal
- **🎯 Contenido:** Flujo (notebook 01) + señal antes/después:
  1. **Winsorización** de outliers (evita que artefactos musculares saturen el espectro).
  2. **Filtro Butterworth** 4º orden (quita offset DC y ruido de alta frecuencia; banda útil 4–40 Hz).
  3. **Normalización** centrada en la media.
- **🗣️ Contexto:** Sin esto, las transformadas "verían" artefactos en vez de actividad neuronal.
- **⭐ Puntos clave:** Explicar el *por qué* de cada paso: winsorizar preserva la serie, el filtro acota la banda, normalizar permite comparar canales.

---

### Diapo 6 — Limpieza de artefactos con ICA ⭐ *(aporte nuevo)*
- **🎯 Contenido:** Notebook **01b**. Problema detectado: el clasificador se apoyaba en canales **frontales/temporales (AF3, F7, T7)**, no en los occipitales del alfa → sospecha de que usaba **artefactos oculares** como atajo.
  - **FastICA** descompone los 14 canales; se identifica **1 componente ocular** por heurística (razón frontal, correlación con "EOG virtual", curtosis).
  - Al removerlo: se elimina **57 % de la varianza frontal** pero solo **4.4 % de la occipital** → se quita el artefacto **preservando el alfa fisiológico**.
- **🗣️ Contexto:** Es el aporte metodológico más fuerte de la rama TF; corrige una validez fisiológica dudosa.
- **⭐ Puntos clave:**
  - Tras la limpieza, el canal dominante en SST pasó de FC5 (frontal) a **O2 (occipital)** → el modelo "mira" donde debe.
  - El rendimiento se **mantuvo y se hizo más estable** (no era solo el artefacto el que clasificaba). Se adopta como mejora canónica.

---

### Diapo 7 — Fase 2: Análisis tiempo-frecuencia y las 4 técnicas
- **🎯 Contenido:**
  - Idea: ver la energía en **tiempo y frecuencia a la vez**; trade-off de resolución (principio de incertidumbre).
  - Técnicas (notebook 02): **STFT** (ventana fija, baseline), **CWT** (multirresolución, la apuesta), **SST** (espectro afilado por synchrosqueezing) y **WPD** (descomposición en sub-bandas).
- **🗣️ Contexto:** Núcleo comparativo. Cada técnica produce una representación 2D de energía espectral → luego features.
- **⭐ Puntos clave:**
  - No hay resolución perfecta en tiempo y frecuencia a la vez → cada técnica resuelve el trade-off distinto (conecta con la hipótesis).
  - Son **4 técnicas comparadas en igualdad de condiciones**, no solo STFT vs. CWT.

---

### Diapo 8 — Extensión: métodos topológicos (GSP y TSP) *(opcional / a futuro)*
- **🎯 Contenido:** Rama "b" del pipeline (notebooks 02b–07b). Las técnicas TF analizan **cada canal por separado**; el cerebro es un sistema **distribuido**.
  - **GSP (Graph Signal Processing):** trata los electrodos como nodos de un grafo y analiza patrones espaciales (interacciones par a par).
  - **TSP (Topological Signal Processing):** captura interacciones de **orden superior** (ciclos/triángulos entre regiones), relevantes en estados de sincronización.
- **🗣️ Contexto:** Extensión exploratoria que mira la **estructura espacial**, no solo cada canal aislado.
- **⭐ Puntos clave:**
  - Honestidad: la maquinaria (EDA 04b, comparación 05b, clasificación 06b/07b) **está implementada pero aún no ejecutada** contra los datos → resultados de clasificación **pendientes**. Presentarlo como línea metodológica y de trabajo futuro, no como resultado cerrado.

---

### Diapo 9 — Fase 3: Representaciones 2D y extracción de features
- **🎯 Contenido:**
  - **Visual:** escalograma ojos-cerrados vs. abiertos; señalar la banda alfa "encendiéndose" al cerrar los ojos.
  - De la imagen 2D al **dataset tabular** (notebook 03): 4 features por canal centradas en alfa (`alpha_abs`, `alpha_rel`, `entropy`, `cog`). **Dos configuraciones:** **A** = todos los canales (~56 features), **B** = solo alfa (~16 features). GSP/TSP usan config única (4 features).
- **🗣️ Contexto:** Momento visual fuerte + cómo se traduce a números que un modelo puede usar.
- **⭐ Puntos clave:**
  - Mostrar con el puntero el aumento de energía en alfa con ojos cerrados.
  - A vs. B responde: *¿conviene solo alfa (hipótesis pura) o toda la información de contexto?*

---

### Diapo 10 — Fase 4: Diseño de clasificación y resultados
- **🎯 Contenido:**
  - **3 clasificadores** (Reg. Logística, SVM-RBF, Random Forest) × técnicas × configs. En la rama TF: 4 técnicas × 2 configs = 24 combos; sumando GSP/TSP llega a **30 combinaciones**. **Split temporal** sin *data leakage*.
  - Mejor modelo (con limpieza ICA): **CWT + Config A + Random Forest → BAcc 0.615 ± 0.065**.
- **🗣️ Contexto:** Comparación justa y sistemática. La CWT lidera, coherente con la hipótesis.
- **⭐ Puntos clave:**
  - **Split temporal** (no aleatorio): mezclar filtraría el futuro e inflaría las métricas.
  - **Honestidad:** BAcc ~0.61 es modesto; los modelos tendían a **favorecer una clase** → hay que ponerlo a prueba (diapo 12).

---

### Diapo 11 — Interpretabilidad: ¿qué canales mandan?
- **🎯 Contenido:** Importancia de features del Random Forest, **antes vs. después de la ICA**. Antes dominaban T7/F7 (temporales/frontales); tras la limpieza, sube el peso de **canales alfa occipitales** (28.7 % → 32.8 %; O2 pasa a dominar en SST).
- **🗣️ Contexto:** Conecta la limpieza ICA con un resultado interpretable y fisiológicamente válido.
- **⭐ Puntos clave:**
  - El alfa "vive" en occipitales (O1/O2/P7/P8). Que tras limpiar el modelo se apoye más en ellos es **evidencia de que ahora clasifica por el ritmo real**, no por artefactos.
  - Residual: T7 sigue pesando → lleva a la siguiente discusión de límites.

---

### Diapo 12 — El giro: evaluación robusta (notebooks 07/07b)
- **🎯 Contenido:**
  - **Problema del split único:** distribución asimétrica (train 45.8 % vs. test 62.9 % clase 0). Un modelo que **siempre diga "ojos abiertos"** saca 62.9 % sin aprender → la accuracy engaña.
  - **Solución:** **Balanced Accuracy** + **cross-validación temporal (TimeSeriesSplit, 5 folds)** + curvas ROC, aplicada a las 6 técnicas.
- **🗣️ Contexto:** Aporte metodológico más maduro del trabajo: cuestionar las propias métricas.
- **⭐ Puntos clave:** BAcc = promedio de sensibilidad y especificidad → corrige el sesgo. TimeSeriesSplit respeta el orden temporal y entrega `media ± std` en vez de una sola estimación.

---

### Diapo 13 — El resultado honesto y sus límites
- **🎯 Contenido:**
  - Con evaluación robusta, el desempeño se estanca en **~0.62–0.67 BAcc** y la **CWT lidera, pero las diferencias entre técnicas TF no son estadísticamente significativas**.
  - Dos límites explorados y documentados honestamente:
    - **Artefacto muscular de T7:** no separable con solo 14 canales (la ICA dañaría alfa real de O1 al intentar removerlo).
    - **Features multibanda** (theta/beta/gamma + IAF): subían la media a 0.670 pero **duplicaban la varianza** → no significativo, **no se adoptó**.
- **🗣️ Contexto:** Cierra el círculo demostrando rigor: se sabe *por qué* el modelo no mejora más.
- **⭐ Puntos clave:**
  - El **techo es de los datos, no del modelo**: un solo sujeto, ~117 s, 231 ventanas con 75 % de solapamiento → pocas muestras *independientes* → alta varianza.
  - Que las diferencias entre técnicas **no sean significativas es en sí un resultado metodológico válido**, no un fracaso.

---

### Diapo 14 — Conclusiones, trabajo futuro y cierre
- **🎯 Contenido:**
  - **Conclusiones:** pipeline reproducible completo; **CWT > STFT** de forma consistente pero **modesta y no significativa**; la **limpieza ICA** mejora la validez fisiológica y la estabilidad; el desempeño (~0.62 BAcc) está limitado por los datos.
  - **Trabajo futuro:** (1) más datos / múltiples sujetos — el único salto real; (2) hardware con más canales y EOG dedicado; (3) ejecutar y cerrar la rama topológica (GSP/TSP); (4) features multibanda y de conectividad/sincronía con regularización; (5) tests estadísticos formales entre técnicas.
  - "Gracias / ¿Preguntas?"
- **🗣️ Contexto:** Síntesis honesta que responde la pregunta de investigación y muestra autocrítica.
- **⭐ Puntos clave:**
  - Responder la pregunta: *"La CWT representa mejor el alfa, pero con las features y datos actuales la ventaja es limitada."*
  - Frase final: *"La lección más valiosa fue aprender a no confiar en una sola métrica y a separar el alfa real de los artefactos."*
  - Tener a mano: 14 canales, 128 Hz, 231 ventanas, hasta 30 combinaciones, mejor BAcc ≈ 0.615.

---

### Notas para el expositor
- **Hilo narrativo:** EEG difícil → ritmo alfa → pregunta/hipótesis → pipeline → limpieza ICA (validez fisiológica) → 4 técnicas TF (+ extensión topológica) → "CWT lidera" → ¿es robusto? → verdad honesta (ventaja no significativa, techo de datos) → qué sigue.
- **Tono:** vender el **rigor**, no un accuracy alto. La limpieza ICA, la evaluación robusta y reconocer los límites valen más que un número bonito.
- **Reparto (4 personas):** (1) diapos 1–3, (2) 4–6, (3) 7–9, (4) 10–14.
- **Si falta tiempo:** saltar diapo 8 (topológicas, aún sin resultados) y comprimir 9 dentro de 7 → ~11 diapos.
- **Back-up (solo si preguntan):** tabla completa de combinaciones (Acc/BAcc/AUC), curvas ROC superpuestas, detalle de la heurística ICA, anexo GSP/EEGraSP (notebook 99).
- **Verificar antes de exponer:** confirmar contra los notebooks las cifras (231 ventanas, BAcc 0.615 ± 0.065, 0.670 ± 0.120 multibanda, % de varianza ICA). La rama topológica aún **no está ejecutada**: no presentar números de GSP/TSP.
