# Challenge 03: Inteligencia Geo-Temporal y de Redes

**TechLogistics S.A.** | Maestría en Ciencia de Datos - EAFIT | Periodo 2026-1
Docente: Jorge Iván Padilla-Buriticá

**Integrantes del equipo:**

| Nombre completo | Cédula |
| --- | --- |
| Samuel Gutiérrez Jaramillo | 1036449975 |
| David Lopera Londoño | 1011392448 |
| Juan Diego Acuña Giraldo | 1020222381 |

## 1. Descripción del proyecto

TechLogistics S.A. (empresa ficticia) tiene dos redes de sensores georreferenciadas
pero desconectadas entre sí: una red agroindustrial (cultivos en el Oriente
Antioqueño) y una red eléctrica nacional. La junta directiva necesita entender:

1. Cómo se propaga el ruido en la red de sensores (Grafos).
2. Dónde se localizan los puntos críticos de calor (Geoespacial).
3. Cuál es el pronóstico de carga (Series de Tiempo).

El proyecto sigue una metodología CRISP-DM organizada en 4 fases, todas resueltas
en un único notebook (`notebooks/challenge03.ipynb`):

| Fase | Contenido | Estado |
|---|---|---|
| 1. Data Understanding y Geo-Visualización | Exploración geoespacial (NDVI/Humedad) + Estacionariedad (ADF/KPSS) | ✅ Completa |
| 2. Procesamiento de Señales | FFT, espectrogramas, filtro Butterworth | ✅ Completa |
| 3. Análisis de Grafos | Grafo de sensores/subestaciones, centralidades | ✅ Completa |
| 4. Modelado y Decisión | Granger (P1), recomendación hídrica (P2), ARIMAX (P3) | ✅ Completa |

## 2. Estructura del repositorio

```
Challenge_03_Data_Science/
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/                           # CSVs originales (clean y noise), intocables
│   └── processed/                     # Series filtradas, diferenciadas, grafo exportado
│
├── notebooks/
│   └── challenge03.ipynb              # Notebook único: Fases 1 a 4 completas
│
├── results/
│   ├── figuras/                       # HTML/PNG de gráficas clave (mapas, PSD, grafos, etc.)
│   ├── figuras_pdf/                   # Versiones estáticas (PNG) usadas en el Informe Técnico
│   ├── diagnostico_estacionariedad_ener.csv   # Tabla ADF/KPSS por variable
│   ├── metricas_centralidad.csv       # Degree Centrality / Betweenness por nodo (Fase 3)
│   └── Challenge_03_Informe_Tecnico.pdf
│
└── docs/
    ├── Challenge_03_Informe_Tecnico.pdf   # Informe ejecutivo final (copia de entrega)
    └── declaracion_uso_IA.md          # Trazabilidad de uso de IA
```

Se eliminaron las carpetas `src/` y `assets/` que estaban en el scaffolding inicial
porque, al resolver todo el análisis en un único notebook (`challenge03.ipynb`), no
llegamos a necesitar módulos de Python separados ni capturas de pantalla aparte —
mantenerlas vacías solo agregaba ruido a la estructura.

## 3. Datos

Dos pares de datasets clean/noise (misma señal, una versión limpia y otra con
ruido gaussiano inyectado, SNR entre 5-12dB):

- `agro_clean.csv` / `agro_noise.csv`: 14 sensores agroindustriales, 2000
  lecturas, variables hídricas (`Agro_1-3`), radiación PAR (`Agro_4`), índices
  bióticos NDVI/Biomasa (`Agro_5-7`), suelo/viento (`Agro_8-10`), más
  `Latitude`, `Longitude`, `Source_Node`, `Target_Node`.
- `ener_clean.csv` / `ener_noise.csv`: red eléctrica nacional (20 subestaciones,
  50 nodos de carga), mercado spot (`Ener_1-3`), generación eólica (`Ener_4`),
  factores macro (`Ener_5-7`), calidad de potencia (`Ener_8-10`), más
  geolocalización de subestaciones.

## 4. Cómo reproducir el notebook

```bash
pip install -r requirements.txt
cd notebooks
jupyter nbconvert --to notebook --execute --inplace challenge03.ipynb
```

El notebook lee los CSV desde `../data/raw/` con rutas relativas, así que debe
ejecutarse desde dentro de `notebooks/`.

## 5. Hallazgos de la Fase 1 (Camilo)

### Tarea 1 — Exploración Geo-Temporal
- 14 sensores agroindustriales, sin nulos ni duplicados.
- Mapa `scatter_mapbox`: color = NDVI (`Agro_5`), tamaño = Humedad (`Agro_1`).
- El análisis de clustering espacial usó inicialmente un promedio agregado de
  distancias, que resultó engañoso por un outlier dentro del grupo. Al
  recalcular con distancia real (Haversine, en metros) y desagregar por
  pares, se confirmó un **clúster compacto entre los nodos 14, 5 y 1**
  (270-644 m entre ellos), mientras que el **nodo 10 es un outlier aislado**
  (864-1,422 m de los otros tres).
- Implicación de negocio: la zona de los nodos 14, 5 y 1 es candidata
  prioritaria para inversión en infraestructura hídrica (pregunta P2, Fase 4).

### Tarea 2 — Estacionariedad y Windowing
- Test ADF + KPSS sobre las 10 variables de `ener_clean.csv`.
- **No Estacionarias, evidencia sólida (ADF y KPSS coinciden):** `Ener_5`,
  `Ener_6`, `Ener_7`.
- **No Estacionarias, caso ambiguo (ADF y KPSS se contradicen, tratadas como
  no estacionarias por precaución metodológica):** `Ener_1`, `Ener_2`,
  `Ener_3`.
- **Estacionarias, evidencia sólida (ADF y KPSS coinciden):** `Ener_4`,
  `Ener_8`, `Ener_9`, `Ener_10`.
- `Ener_5` (Costo del Gas) muestra comportamiento de **Drift** (pendiente
  positiva sostenida en la media móvil de ventana 50), no random walk puro.
- Esta clasificación se usó consistentemente en las Fases 2-4: diferenciar
  `Ener_1-3` y `Ener_5-7` antes de correlación de Pearson o modelos ARIMA/ARIMAX.

## 6. Hallazgos de la Fase 2 (Samuel)

### Tarea 3 — Análisis Espectral (FFT) y Espectrogramas
- Supuestos de muestreo: `fs=1`, frecuencias en ciclos/muestra (Nyquist = 0.5).
- PSD (Welch) sobre `Ener_4` (generación eólica) en versión clean vs noise: la
  señal concentra ~100% de su potencia en la banda baja [0, 0.05), mientras que
  el ruido inyectado es de banda ancha (~61% de la potencia del residual está en
  [0.2, 0.5)).
- SNR estimado de `Ener_4` ≈ 8.5 dB, dentro del rango inyectado (5-12 dB).
- Los espectrogramas (STFT, ventana Hann 256, 50% solapamiento) confirman que la
  banda de ruido es la alta, separada del contenido espectral de la señal.

### Tarea 4 — Filtrado y Reconstrucción
- Butterworth pasa-bajo de orden 4 con cutoff = 0.15 ciclos/muestra, aplicado con
  `filtfilt` (fase cero) sobre `Agro_3_noise` (Humedad Relativa).
- RMSE de reconstrucción contra la original: 3.34 (ruidosa) → 1.85 (filtrada),
  una reducción de ~45%. La serie filtrada conserva la varianza (std 11.61 vs
  11.56).
- Pronóstico AR one-step-ahead: la filtrada supera a la ruidosa en todos los
  órdenes probados (AR(1): 4.53 → 0.82, AR(2): 3.98 → 2.15, AR(3): 3.73 → 2.41).
- Coeficientes AR(2): hallazgo mixto — la ruidosa queda casi igual a la clean
  (~-4% a -5% en los lags), mientras la filtrada infla Lag1 (+248%) y cambia el
  signo de Lag2. El filtrado mejora el error de pronóstico, pero no recupera los
  coeficientes estructurales del modelo.

## 7. Hallazgos de la Fase 3 (David)

### Tarea 5 — Construcción de la Red de Sensores/Subestaciones
- Grafos dirigidos por dataset (AGRO: 29 nodos/210 aristas; ENER: 70 nodos/865
  aristas), ponderados por volumen de lecturas de telemetría.
- Ambas redes resultaron ser **bipartitas de 2 capas**: `Source_Node` y
  `Target_Node` son conjuntos disjuntos, así que todo camino tiene longitud 1 y
  **Betweenness Centrality da 0 en todos los nodos de ambas redes** — hallazgo
  estructural, no un error de cálculo. Se verificó con un segundo método
  (Betweenness sobre la versión no dirigida) para confirmar robustez.
- En AGRO, la red es bipartita **completa** (densidad 100%), así que ni la
  Degree Centrality distingue nodos — el cuello de botella se determinó por
  volumen de telemetría: **Nodo 10**.
- En ENER, tres métricas independientes (Degree Centrality, grado ponderado, y
  Betweenness no dirigida) coinciden en el mismo nodo: **Nodo 119**, que
  concentra el 6.0% del tráfico total y alcanza el 98% de las subestaciones
  destino.

## 8. Hallazgos de la Fase 4 (Juan Diego + equipo)

### P1 — Causalidad y Redes
- Causalidad de Granger entre `Ener_10` (Factor de Potencia) y `Ener_9`
  (Voltaje): una señal inicial en lags 4-5 (p≈0.02) no sobrevivió la
  verificación independiente con selección de orden VAR (AIC/BIC eligen
  lag=0) — se trató como falso positivo por comparaciones múltiples.
  **Conclusión: no hay evidencia robusta de causalidad.**
- Impacto de red hipotético: el Nodo 119 (mayor centralidad, Fase 3)
  concentra el mayor riesgo sistémico si esa causalidad existiera, dado que
  la red no tiene nodos intermedios que contengan la propagación de una falla.

### P2 — Optimización Geo-Agrónoma
- Filtrado del jitter GPS de `agro_noise` (AWGN de media cero): el error baja
  de ~900 m por lectura a ~74 m al promediar por sensor.
- Confirma el clúster de NDVI bajo de la Fase 1 (nodos 14, 5, 1) y aísla al
  nodo 10. El proxy de pendiente (varianza del viento, `Agro_10`) no explica
  el NDVI bajo en general (r≈0.05), pero sí identifica al nodo 10 como el de
  mayor exposición eólica de la red.
- Recomendación en dos zonas: Zona A (nodos 14, 5, 1) prioridad alta, un solo
  frente de obra hídrica; Zona B (nodo 10) monitoreo antes de invertir.

### P3 — Analítica Predictiva
- ARIMAX para la Demanda (`Ener_1`) con Temperatura (`Ener_3`) y Centralidad
  del nodo de origen (Fase 3) como exógenas, `d=1` (consistente con la
  clasificación de la Fase 1).
- **La Centralidad del nodo NO mejora el modelo**: AIC pasa de 8,565.50 (solo
  Temperatura) a 8,569.41 (agregando Centralidad); el coeficiente no es
  significativo (p=0.066) y la correlación cruda es prácticamente nula
  (r=0.015). La estructura de red es valiosa para el análisis de riesgo de
  propagación de fallas (P1), pero no para este tipo de pronóstico.

## 9. Informe Técnico

El documento ejecutivo con las 5 secciones del checklist (contexto, qué se
analizó, hallazgos por fase con evidencia gráfica, las 4 Preguntas de
Validación del checklist respondidas explícitamente, y plan de acción
priorizado) está en [`docs/Challenge_03_Informe_Tecnico.pdf`](docs/Challenge_03_Informe_Tecnico.pdf).

## 10. Declaración de uso de IA

Ver [`docs/declaracion_uso_IA.md`](docs/declaracion_uso_IA.md).

