# Challenge 03: Inteligencia Geo-Temporal y de Redes

**TechLogistics S.A.** | Maestría en Ciencia de Datos - EAFIT | Periodo 2026-2
Docente: Jorge Iván Padilla-Buriticá

## 1. Descripción del proyecto

TechLogistics S.A. (empresa ficticia) tiene dos redes de sensores georreferenciadas
pero desconectadas entre sí: una red agroindustrial (cultivos en el Oriente
Antioqueño) y una red eléctrica nacional. La junta directiva necesita entender:

1. Cómo se propaga el ruido en la red de sensores (Grafos).
2. Dónde se localizan los puntos críticos de calor (Geoespacial).
3. Cuál es el pronóstico de carga (Series de Tiempo).

El proyecto sigue una metodología CRISP-DM organizada en 4 fases:

| Fase | Contenido | Estado |
|---|---|---|
| 1. Data Understanding y Geo-Visualización | Exploración geoespacial (NDVI/Humedad) + Estacionariedad (ADF/KPSS) | ✅ Completa |
| 2. Procesamiento de Señales | FFT, espectrogramas, filtro Butterworth | ⏳ Pendiente |
| 3. Análisis de Grafos | Grafo de sensores/subestaciones, centralidades | ⏳ Pendiente |
| 4. Modelado y Decisión | Granger, recomendación hídrica, ARIMAX | ⏳ Pendiente |

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
│   └── challenge03_fase1.ipynb        # Fase 1: geo-visualización + estacionariedad
│
├── src/
│   ├── __init__.py
│   ├── data_loader.py                 # Carga de CSVs
│   ├── stationarity.py                # ADF, KPSS, diferenciación
│   ├── signal_processing.py           # FFT, espectrogramas, filtro Butterworth, RMSE
│   ├── graph_analysis.py              # Construcción del grafo, centralidades
│   └── geo_viz.py                     # scatter_mapbox, mapas de calor
│
├── results/
│   ├── figuras/                       # HTML/PNG de gráficas clave
│   └── diagnostico_estacionariedad_ener.csv   # Tabla ADF/KPSS por variable
│
├── docs/
│   ├── Challenge_03_Informe_Tecnico.pdf   # Informe ejecutivo final
│   └── declaracion_uso_IA.md          # Trazabilidad de uso de IA
│
└── assets/
    └── screenshots/                   # Capturas de mapas, grafos, espectrogramas
```

## 3. Datos

Dos pares de datasets clean/noise (misma señal, una versión limpia y otra con
ruido gaussiano inyectado, SNR entre 5-12dB):

- `agro_clean.csv` / `agro_noise.csv`: 14 sensores agroindustriales, 2000
  lecturas, variables hídricas (`Agro_1-3`), radiación PAR (`Agro_4`), índices
  bióticos NDVI/Biomasa (`Agro_5-7`), suelo/viento (`Agro_8-10`), más
  `Latitude`, `Longitude`, `Source_Node`, `Target_Node`.
- `ener_clean.csv` / `ener_noise.csv`: red eléctrica nacional, mercado spot
  (`Ener_1-3`), generación eólica (`Ener_4`), factores macro (`Ener_5-7`),
  calidad de potencia (`Ener_8-10`), más geolocalización de subestaciones.

## 4. Cómo reproducir la Fase 1

```bash
pip install -r requirements.txt
cd notebooks
jupyter nbconvert --to notebook --execute --inplace challenge03_fase1.ipynb
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
- Esta clasificación debe usarse en las Fases 2-4: diferenciar `Ener_1-3` y
  `Ener_5-7` antes de correlación de Pearson o modelos ARIMA/ARIMAX.

## 6. Equipo


## 7. Declaración de uso de IA

Ver [`docs/declaracion_uso_IA.md`](docs/declaracion_uso_IA.md).

## 8. Plazo de entrega

07 de febrero de 2026 (23:59 COT).