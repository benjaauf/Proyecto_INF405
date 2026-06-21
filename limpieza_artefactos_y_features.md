# Limpieza de artefactos y enriquecimiento de features — exploración, limitaciones y trabajo futuro

Documento complementario al proyecto INF405 (comparación de técnicas tiempo-frecuencia
para la caracterización del ritmo alfa en EEG, dataset UCI EEG Eye State). Resume tres
procesos exploratorios realizados sobre el pipeline de clasificación, las limitaciones
encontradas y las posibles líneas de trabajo futuro.

---

## Contexto

El clasificador distingue **ojos abiertos (0) vs ojos cerrados (1)** a partir de features
extraídas de cuatro representaciones tiempo-frecuencia (STFT, CWT, SST, WPD). El análisis
de importancia del Random Forest (notebooks 06/07) reveló un problema: los canales que más
pesaban en la clasificación eran **frontales y temporales** (AF3, F7, T7), no los
occipitales (O1, O2, P7, P8) donde fisiológicamente debería dominar el ritmo alfa.

La hipótesis fue que el clasificador estaba usando **artefactos oculares y musculares**
—que están físicamente correlacionados con el estado de los ojos— como atajo, en lugar del
ritmo alfa real. Los tres procesos siguientes buscaron corregir esto y mejorar el modelo.

---

## Proceso 1 — Limpieza de artefactos oculares con ICA  ✅ adoptado

**Qué se hizo.** Se añadió el notebook `01b_LIMPIEZA_ICA.ipynb` entre el acondicionamiento
(01) y la transformación TF (02). Usa **FastICA** (sklearn) para descomponer los 14 canales
en 14 componentes independientes e identificar los oculares mediante una heurística
combinada (no hay canal EOG dedicado):

1. **Razón frontal** — fracción del peso espacial del componente en canales cercanos a los
   ojos (AF3, AF4, F7, F8).
2. **Correlación frontal** — correlación de la serie del componente con la media de esos
   canales (un "EOG virtual").
3. **Curtosis** — los parpadeos/sacadas son picos no-gaussianos.

**Resultado.** Se identificó **un único componente ocular** (criterio conservador), de
firma clara: razón frontal 0.51, correlación 0.83 con el EOG virtual, curtosis 11.6. Al
removerlo y reconstruir la señal:

| Grupo de canales | Varianza removida |
|---|---|
| Frontales (AF3, AF4, F7, F8) | **57 %** promedio (AF3: 82 %) |
| Occipitales (O1, O2, P7, P8) | **4.4 %** promedio |

Es decir, se eliminó el artefacto concentrado en los frontales **preservando casi intacto
el ritmo alfa occipital** — el resultado deseado.

**Efecto en la clasificación.** El rendimiento se **mantuvo** (no colapsó), y mejoró en lo
cualitativo:

- El mejor modelo individual pasó de SST+ConfigA+SVM `0.607 ± 0.131` (sin limpieza) a
  **CWT+ConfigA+RF `0.615 ± 0.065`** (con limpieza) — media ligeramente mayor y, sobre
  todo, **el doble de estable**.
- En SST, el canal dominante del Random Forest pasó de FC5 (frontal-central) a **O2
  (occipital)**, y el porcentaje de importancia en canales alfa subió (28.7 % → 32.8 %).

**Conclusión.** La limpieza ocular se adopta como mejora canónica: el clasificador se apoya
más en el ritmo alfa fisiológico y el mejor modelo es más estable. Genera
`data/eeg_eye_state_clean_ica.csv` y las features `data/features/features_*_ica.csv`.

---

## Proceso 2 — Limpieza del artefacto muscular de T7  ⚠️ no viable (limitación)

**Qué se intentó.** Tras la limpieza ocular, el canal **T7 (temporal)** seguía dominando
la importancia del clasificador. Se intentó extender la ICA para remover ese artefacto
muscular (EMG), buscando un componente que fuera **temporal-dominante (T7/T8)** *y* con
**espectro dominado por alta frecuencia (>20 Hz)** —la firma típica del EMG.

**Por qué no funcionó.**

1. **No existe un componente muscular aislable.** Ningún componente cumplía ambos criterios:
   el más temporal (dominante en T7, curtosis 105) tenía solo ~36 % de su potencia sobre
   20 Hz → es un **transitorio** esporádico (movimiento / pop de electrodo), no EMG
   sostenido. El artefacto muscular son muchas unidades motoras distribuidas que, con solo
   **14 canales**, la ICA no logra separar en un componente propio (limitación conocida de
   la ICA con pocos electrodos).

2. **Removerlo sería contraproducente.** Ese componente T7-dominante mezcla el artefacto de
   T7 con alfa de O1. Eliminarlo quitaría solo ~10 % de la varianza de T7 pero **~13 % del
   alfa genuino de O1** — se dañaría un canal alfa real para apenas tocar el artefacto.

**Conclusión.** Se conserva únicamente la remoción ocular. El residual de T7 queda
documentado como **limitación metodológica** atribuible al bajo número de canales del
Emotiv EPOC. El diagnóstico está incluido en `01b_LIMPIEZA_ICA.ipynb`.

---

## Proceso 3 — Enriquecimiento de features multibanda  ⚠️ no adoptado

**Qué se intentó.** Las features originales son 4 por canal, todas centradas en alfa
(`alpha_abs`, `alpha_rel`, `entropy`, `cog`). Se probó un set enriquecido de **9 features
por canal**: potencias relativas de **theta, alfa, beta, gamma**, ratio **alfa/beta**,
**IAF** (frecuencia del pico alfa) y cog. Se evaluó con **selección de features dentro del
CV** (SelectKBest) para mitigar la alta dimensionalidad.

**Resultado.**

| Configuración | BAcc CV |
|---|---|
| Limpieza ocular + 4 features (sin selección) | **0.615 ± 0.065** |
| Limpieza ocular + 4 features + selección | 0.648 ± 0.111 |
| Limpieza ocular + 9 features multibanda + selección | **0.670 ± 0.120** |

La media subió (0.615 → 0.670), pero:

- La ganancia (~0.055) es de **~1 error estándar → no es estadísticamente significativa**
  con solo 5 folds.
- La **varianza se duplicó** (±0.065 → ±0.120): el modelo enriquecido podría rendir ~0.55
  en un segmento nuevo, peor que el *piso* del modelo simple.
- El aumento de varianza proviene del **estrés de dimensionalidad** (126 features con folds
  que entrenan con ~40 muestras), no de una mejora real: el aparato adicional ajusta ruido.

**Conclusión.** No se adopta. El modelo más simple (limpieza ocular + 4 features
originales) es el más estable, reproducible y honesto. Las features multibanda se dejan
documentadas como avenida explorada.

---

## Limitaciones generales (el techo de los datos)

La causa de fondo de que el rendimiento se estanque en **~0.62–0.67 BAcc** no es el
modelado, sino los datos:

- **Un solo sujeto** y un único registro de **~117 segundos**.
- **231 ventanas** con **75 % de solapamiento** → las muestras *independientes* reales son
  muchas menos, lo que produce la alta varianza entre folds (±0.06 a ±0.24).
- **14 canales sin electrodo EOG dedicado** → limita tanto la identificación de artefactos
  (Proceso 1) como su separación (Proceso 2).
- **Artefactos correlacionados con el label**: como los parpadeos/sacadas solo ocurren con
  ojos abiertos, los artefactos son informativos para la tarea; removerlos mejora la
  validez fisiológica pero no necesariamente la accuracy cruda.

Con esta varianza, **las diferencias entre técnicas TF (STFT/CWT/SST/WPD) y entre
configuraciones de features no son estadísticamente significativas** — lo cual es en sí un
resultado metodológico válido.

---

## Trabajo futuro

Ordenado por impacto esperado:

1. **Más datos (el único salto real).** Registros de **múltiples sujetos** y sesiones más
   largas romperían el techo de varianza. Es la limitación dominante; ningún truco de
   modelado la sustituye.

2. **Hardware con más canales y EOG dedicado.** Un montaje de 32/64 canales con electrodos
   EOG permitiría: (a) ICA mucho más efectiva, (b) identificación de componentes oculares
   por referencia directa en vez de heurística, y (c) separación del EMG temporal que aquí
   no fue posible (Proceso 2).

3. **Supresión quirúrgica de transitorios en T7/T8.** Como alternativa a la ICA para el
   artefacto temporal: detección y interpolación de picos *por canal* (solo T7/T8), sin
   tocar O1. Es más ad-hoc pero evitaría el daño colateral al alfa occipital.

4. **Features multibanda con regularización adecuada.** Con más muestras, el set enriquecido
   (theta/alfa/beta/gamma + ratios + IAF) sí podría aportar sin disparar la varianza.
   Incluir también la **banda delta** (requiere extender el rango TF del notebook 02, hoy
   limitado a 4–40 Hz).

5. **Features de conectividad / sincronía.** Coherencia entre canales occipitales (O1–O2),
   aprovechando la estructura espacial que el análisis GSP del notebook 02 ya insinuaba. El
   ritmo alfa con ojos cerrados es sincronizado, por lo que la coherencia podría ser un
   marcador robusto.

6. **Tests estadísticos formales entre técnicas.** Reportar intervalos de confianza y
   pruebas de significancia (p. ej. test de permutación) para sustentar formalmente que las
   diferencias entre STFT/CWT/SST/WPD no son significativas con el tamaño muestral actual.

---

## Archivos relacionados

- `01b_LIMPIEZA_ICA.ipynb` — limpieza ocular con ICA + diagnóstico de la limitación de T7.
- `data/eeg_eye_state_clean_ica.csv` — señal con artefacto ocular removido.
- `data/features/features_*_ica.csv` — features (4/canal) desde la señal limpia.
- `06_CLASIFICACION.ipynb`, `07_EVALUACION_ROBUSTA.ipynb` — clasificación y evaluación robusta.
