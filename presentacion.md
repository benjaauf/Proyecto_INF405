# Presentación — Proyecto INF405
## Comparación de técnicas tiempo-frecuencia para la caracterización del ritmo alfa en EEG (ojos abiertos vs. cerrados)

**Integrantes:** Matías Peñaloza · Benjamín Ulloa · Alfredo Iturra · Juan Suárez
**Curso:** Procesamiento de Señales e Imágenes (INF405)

---

> **Formato:** 12 diapositivas (1 portada + 10 de contenido + 1 cierre). ~15 min, ~1–1.5 min por diapo.
> Cada diapo trae: **🎯 Contenido** (qué se muestra) · **🗣️ Contexto** (de qué se habla) · **⭐ Puntos clave** (lo que sí o sí hay que decir).

---

### Diapo 1 — Portada
- **🎯 Contenido:** Título, integrantes, curso, fecha. Imagen de fondo (EEG / electrodos).
- **🗣️ Contexto:** Primera impresión; el tema debe quedar claro al instante.
- **⭐ Puntos clave:** Frase gancho: *"¿Qué técnica tiempo-frecuencia describe mejor el ritmo alfa para clasificar si una persona tiene los ojos abiertos o cerrados?"*

---

### Diapo 2 — Problema, fenómeno y pregunta *(todo el "por qué" en una)*
- **🎯 Contenido:** Tres ideas:
  1. El EEG es difícil: señales de baja amplitud, **no estacionarias** y con muchos artefactos.
  2. El **ritmo alfa** (~8–13 Hz) **aumenta con ojos cerrados** y se bloquea al abrirlos → da la etiqueta de clasificación.
  3. **Pregunta:** ¿qué técnica tiempo-frecuencia representa mejor el alfa y permite clasificar mejor el estado ocular?
- **🗣️ Contexto:** Junta motivación + marco fisiológico + pregunta de investigación. Es la base de todo.
- **⭐ Puntos clave:**
  - "No estacionaria" = razón de ser del análisis tiempo-frecuencia.
  - Problema de **clasificación binaria**: clase 0 = ojos abiertos, clase 1 = ojos cerrados.

---

### Diapo 3 — Hipótesis y objetivos
- **🎯 Contenido:**
  - **Hipótesis:** las **wavelets** (multirresolución) representarían mejor la dinámica no estacionaria del alfa que la STFT, mejorando la clasificación.
  - **4 objetivos** = 4 fases: acondicionar → transformar TF → extraer features → clasificar y comparar.
- **🗣️ Contexto:** La hipótesis es la vara con la que mediremos los resultados; los objetivos anticipan el pipeline.
- **⭐ Puntos clave:** Dejar explícita la apuesta ("creemos que ganan las wavelets") para luego contrastarla con honestidad.

---

### Diapo 4 — Datos y pipeline
- **🎯 Contenido:**
  - **Dataset UCI EEG Eye State** (Roesler 2013): 14 canales, 128 Hz, sesión de 117 s, etiqueta validada por video; tras preprocesar quedan **236 ventanas**.
  - **Diagrama del pipeline:** `01 Acondicionamiento → 02 Transformación TF → 03 Features → 04/05 EDA → 06 Clasificación → 07 Evaluación robusta`.
- **🗣️ Contexto:** El "mapa" del proyecto. Si recuerdan una sola diapo, que sea esta.
- **⭐ Puntos clave:**
  - Datos públicos y pre-validados → reproducibilidad. **Un solo sujeto/sesión** → ojo con la generalización (limitación posterior).
  - Stack: SciPy, PyWavelets, scikit-learn.

---

### Diapo 5 — Fase 1: Acondicionamiento de la señal
- **🎯 Contenido:** Flujo (notebook 01) + señal antes/después:
  1. **Winsorización** de outliers (evita que artefactos musculares saturen el espectro).
  2. **Filtro Butterworth** 4º orden (quita offset DC y ruido de alta frecuencia).
  3. **Normalización** centrada en la media. (Limpieza de artefactos basada en **ICA**.)
- **🗣️ Contexto:** Sin esto, las transformadas "verían" artefactos en vez de actividad neuronal.
- **⭐ Puntos clave:** Explicar el *por qué* de cada paso: winsorizar preserva la serie, el filtro acota la banda útil, normalizar permite comparar canales.

---

### Diapo 6 — Fase 2: Análisis tiempo-frecuencia y técnicas
- **🎯 Contenido:**
  - Idea: ver la energía en **tiempo y frecuencia a la vez**; trade-off de resolución (principio de incertidumbre).
  - Técnicas (notebook 02): **STFT** (ventana fija, baseline), **CWT** (multirresolución, la apuesta), **SST** (espectro afilado) y **WPD** (sub-bandas). *GSP/EEGraSP* como anexo.
- **🗣️ Contexto:** Núcleo comparativo del proyecto. Cada técnica produce una representación 2D de energía espectral.
- **⭐ Puntos clave:**
  - No hay resolución perfecta en tiempo y frecuencia a la vez → cada técnica resuelve el trade-off distinto (conecta con la hipótesis).
  - La comparación de clasificación se centró en **STFT vs. CWT** (ventana fija vs. multirresolución).

---

### Diapo 7 — Fase 2/3: Representaciones 2D y extracción de features
- **🎯 Contenido:**
  - **Visual:** escalograma ojos-cerrados vs. abiertos; señalar la banda alfa "encendiéndose" al cerrar los ojos.
  - De la imagen 2D al **dataset tabular** (notebook 03): potencia/energía en banda alfa. **Dos configuraciones:** **A** = todos los canales (~56 features), **B** = solo alfa (~16 features).
- **🗣️ Contexto:** Momento visual fuerte + cómo se traduce a números que un modelo puede usar.
- **⭐ Puntos clave:**
  - Mostrar con el puntero el aumento de energía en alfa con ojos cerrados.
  - A vs. B responde: *¿conviene solo alfa (hipótesis pura) o toda la información de contexto?*

---

### Diapo 8 — Fase 4: Diseño de clasificación y resultados
- **🎯 Contenido:**
  - **3 clasificadores** (Reg. Logística, SVM-RBF, Random Forest) × **2 técnicas** × **2 configs** = **12 combinaciones**. **Split temporal** sin *data leakage*.
  - Mejor del split único: **CWT + Config B (16 feat.) + SVM → Acc 0.603, AUC 0.621**. Matrices de confusión.
- **🗣️ Contexto:** Comparación justa y sistemática. A primera vista, la hipótesis "gana".
- **⭐ Puntos clave:**
  - **Split temporal** (no aleatorio): mezclar filtraría el futuro e inflaría las métricas.
  - **Honestidad:** Acc ~0.60 es bajo y los modelos **favorecieron una clase** → hay que ponerlo a prueba (diapo 10).

---

### Diapo 9 — Interpretabilidad: ¿qué canales mandan?
- **🎯 Contenido:** Importancia de features del Random Forest: dominan **T7 y F7** (temporales/frontales), **no** los occipitales O1/O2 esperados.
- **🗣️ Contexto:** Hallazgo inesperado y muy interpretable; actividad incorporada para enriquecer el análisis.
- **⭐ Puntos clave:** El alfa "vive" en occipitales (O1/O2/P7/P8). Que dominen frontales/temporales sugiere que la **CWT podría estar capturando artefactos** (parpadeos/músculo), no solo alfa fisiológico → matiza su "victoria".

---

### Diapo 10 — El giro: evaluación robusta (notebook 07)
- **🎯 Contenido:**
  - **Problema del split único:** distribución asimétrica (train 45.8% vs. test 62.9% clase 0). Con 62.9%, un modelo que **siempre diga "ojos abiertos"** saca 62.9% sin aprender → la accuracy engaña.
  - **Solución:** **Balanced Accuracy** + **cross-validación temporal (TimeSeriesSplit, 5 folds)** + curvas ROC.
- **🗣️ Contexto:** Aporte metodológico más maduro del trabajo: cuestionar las propias métricas.
- **⭐ Puntos clave:** BAcc = promedio de sensibilidad y especificidad → corrige el sesgo. TimeSeriesSplit respeta el orden temporal y reduce la varianza de una sola estimación.

---

### Diapo 11 — El resultado honesto
- **🎯 Contenido:**
  - El "ganador" del nb.06 (CWT-B + SVM) resultó **inestable** en CV (**std ± 0.188**).
  - La combinación **más robusta** fue **CWT + Config A + SVM**.
  - **BAcc promedio ≈ 0.59** → apenas sobre el azar (0.5).
- **🗣️ Contexto:** Cierra el círculo con la hipótesis demostrando rigor: el ganador cambió al evaluar bien.
- **⭐ Puntos clave:**
  - **CWT sigue > STFT**, pero la mejor config cambió (B→A) y el margen es pequeño → ventaja **real pero modesta**.
  - BAcc ~0.59 es un resultado **honesto, no decepcionante**: las features actuales tienen **poder discriminativo limitado**.
  - Mensaje: *"evaluar mal nos haría reportar 0.60 con confianza; evaluar bien nos dice la verdad."*

---

### Diapo 12 — Conclusiones, limitaciones y cierre
- **🎯 Contenido:**
  - **Conclusiones:** pipeline reproducible completo; **CWT > STFT** consistente pero modesto; desempeño global cercano al azar (BAcc ≈ 0.59); dominan canales frontales/temporales (posibles artefactos).
  - **Limitaciones / futuro:** un solo sujeto y pocas muestras (236); revisar que la potencia alfa se calcule estrictamente en 8–13 Hz; reducir solapamiento de ventanas; sumar SST/WPD/GSP a la comparación formal; más sujetos.
  - "Gracias / ¿Preguntas?"
- **🗣️ Contexto:** Síntesis honesta que responde la pregunta de investigación y muestra autocrítica.
- **⭐ Puntos clave:**
  - Responder la pregunta: *"La CWT es la que mejor representa el alfa para clasificar, pero la mejora es limitada con las features actuales."*
  - Frase final: *"La wavelet representa mejor el alfa, pero la lección más valiosa fue aprender a no confiar en una sola métrica."*
  - Tener a mano: 14 canales, 128 Hz, 236 ventanas, 12 combinaciones, mejor BAcc(CV) ≈ 0.59.

### Notas para el expositor
- **Hilo narrativo:** EEG difícil → ritmo alfa → pregunta/hipótesis → pipeline → "CWT gana" → ¿es robusto? → verdad honesta (ventaja modesta, BAcc≈0.59) → qué sigue.
- **Tono:** vender el **rigor**, no un accuracy alto. En contexto académico, la evaluación robusta vale más que un número bonito.
- **Reparto (4 personas):** (1) diapos 1–3, (2) 4–6, (3) 7–9, (4) 10–12.
- **Back-up (no presentar, solo si preguntan):** tabla completa de las 12 combinaciones (Acc/BAcc/AUC), curvas ROC superpuestas, detalle del anexo GSP/EEGraSP.
- **Verificar antes de exponer:** confirmar contra los notebooks 06/07 las cifras (236 ventanas, ~56/~16 features, std ±0.188, BAcc≈0.59) antes de fijarlas en las diapos.
