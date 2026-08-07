# Declaración de uso de Inteligencia Artificial

**Challenge 03 — Inteligencia Geo-Temporal y de Redes (TechLogistics S.A.)**

**Equipo:** Camilo, Samuel Gutiérrez Jaramillo, David Lopera Londoño, Juan Diego Acuña Giraldo

**Herramienta usada:** Claude (Anthropic) para la Fase 1, Fase 3, Fase 4, la reestructuración
final del notebook y el Informe Técnico; opencode para la Fase 2.

## Resumen

Usamos IA generativa como apoyo en cinco frentes: **(1)** definir la estructura del repositorio,
**(2)** construir el notebook de la Fase 1 (exploración geoespacial y estacionariedad), **(3)**
las Fases 2, 3 y 4 (procesamiento de señales, grafos, y modelado de decisión), **(4)** reescribir
el notebook completo con un tono más orientado a negocio y una estructura de encabezados
consistente, y **(5)** construir el Informe Técnico en PDF a partir de los resultados ya
generados por el pipeline. En varios puntos, correr el código generado contra los datos reales
sacó a la luz errores que no eran evidentes leyendo el código a simple vista (detallados en la
sección "Auditoría y corrección de errores" más abajo) — esa verificación fue nuestra, no algo
que la IA hiciera de forma autónoma.

## Contexto que se le dio a la IA

Se compartió el caso del Challenge 03 (el enunciado del taller) y una muestra de los datos reales
del proyecto (`agro_clean.csv`, `agro_noise.csv`, `ener_clean.csv`, `ener_noise.csv`), nada más —
el código y las decisiones de análisis se fueron ajustando sobre la marcha a lo que esos datos
mostraban, no a supuestos genéricos.

## Fase 0 — Estructura del repositorio

- *"Dame los comandos para crear la estructura de carpetas del repo: data/raw, data/processed,
  notebooks, results/figuras, docs."*
- *"Ayúdame a decidir dónde va el informe técnico dentro de esa estructura."*

## Fase 1 — Data Understanding y Geo-Visualización (Camilo)

- *"Dame el código para cargar `agro_clean.csv` y graficar un `scatter_mapbox` con color por
  NDVI (`Agro_5`) y tamaño por Humedad (`Agro_1`)."*
- *"Ayúdame a calcular si los sensores de menor NDVI están agrupados geográficamente, comparando
  distancias entre ellos."*
- *"Estoy usando distancia euclidiana sobre latitud/longitud para eso, ¿es correcto o debería
  usar otra fórmula?"* — llevó a cambiar a la fórmula de **Haversine** (distancia real en metros).
- *"Dame el código para aplicar ADF y KPSS a las 10 variables de `ener_clean.csv` y arma una
  tabla comparando si ambos tests coinciden."*
- *"¿Cómo decido qué hacer con las series donde ADF y KPSS no coinciden?"* — se estableció el
  criterio de tratarlas como **no estacionarias por precaución**.
- *"Ayúdame a graficar la media móvil de las series no estacionarias sin que las de menor escala
  se vean planas por comparación con las de mayor magnitud."*
- *"Dame el código para medir si `Ener_5` tiene comportamiento de Drift o de random walk, con la
  pendiente de la media móvil."*

## Fase 2 — Procesamiento de Señales y Filtrado (Samuel)

- *"Dame el código para calcular la densidad espectral de potencia (Welch) de `Ener_4`,
  comparando la versión clean contra la noise."*
- *"Ayúdame a construir un espectrograma (STFT) para ver cómo cambia el espectro en el tiempo."*
- *"Dame el código para un filtro Butterworth pasa-bajo con `filtfilt`, aplicado a
  `Agro_3_noise`."*
- *"Ayúdame a calcular el RMSE de la serie filtrada contra la original, y a comparar el error de
  pronóstico de un modelo AR entre la versión clean, ruidosa y filtrada."*
- *"Estoy comparando los coeficientes AR de las tres versiones con un MAE global, ¿tiene sentido
  promediar así?"* — llevó a corregir la comparación porque mezclaba la constante con los
  coeficientes de rezago en escalas distintas.

## Fase 3 — Análisis de Grafos y Topología de Red (David)

- *"Dame el código para construir un grafo dirigido con NetworkX usando `Source_Node` y
  `Target_Node`, ponderado por número de lecturas de telemetría."*
- *"Dame el código para calcular Degree Centrality y Betweenness Centrality sobre ese grafo."*
- *"Me está dando Betweenness = 0 en todos los nodos, ¿es un error o hay una razón
  estructural?"* — llevó a investigar si la red tenía nodos intermedios, y a confirmar que era
  una **red bipartita de 2 capas**.
- *"Ayúdame a verificar ese resultado con otro método antes de darlo por definitivo"* — se
  calculó Betweenness también sobre la versión no dirigida del grafo.
- *"Como esa métrica no distingue nodos en la red AGRO, dame el código para usar el volumen
  ponderado de telemetría como criterio para encontrar el nodo cuello de botella."*

## Fase 4, P1 y P2 (Juan Diego)

- *"Dame el código para correr Causalidad de Granger entre `Ener_10` y `Ener_9` en ambas
  direcciones, con varios rezagos."*
- *"Me da significativo en los lags 4 y 5 pero no antes, ¿cómo verifico si es una relación real o
  una casualidad estadística?"* — llevó a correr una selección de orden de un modelo VAR
  (AIC/BIC) como chequeo independiente, que **no encontró estructura real**.
- *"Dame el código para filtrar el ruido de las coordenadas GPS de `agro_noise` promediando por
  sensor."*
- *"El dataset no mide pendiente directamente, ayúdame a aproximarla con alguna variable
  disponible"* — se usó la varianza del viento (`Agro_10`) como proxy, documentado como
  **supuesto**, no como hallazgo confirmado.

## Reestructuración del notebook completo y P3 (Camilo)

- *"Antes de ayudarme con el P3, revisa el notebook completo de mis compañeros para que uses el
  mismo criterio y no repitas cálculos que ya hicieron."*
- *"Dame el código para un ARIMAX de la Demanda (`Ener_1`) usando Temperatura y la Centralidad
  del nodo de origen como variables exógenas, y compara el AIC con y sin la centralidad."*
- *"Reescribe las celdas de texto de las Fases 2, 3 y 4 para que expliquen los resultados en
  términos de negocio antes de entrar en el método técnico, igual que hicimos en la Fase 1."*
- *"Agrega un título general al notebook con un índice de las 4 fases, y ordena los encabezados
  de forma consistente."*
- *"Revisa si las 4 preguntas de validación del checklist ya están respondidas en el notebook."*

## Informe Técnico (PDF)

- *"Ayúdame a armar el informe técnico en PDF con reportlab: portada, contexto del encargo, qué
  se analizó, hallazgos por fase con las figuras del notebook, y plan de acción."*
- *"Ajusta el documento a un formato más estándar: Times-Roman, sin colores de marca, menos
  negrilla en el cuerpo del texto."*

## Auditoría y corrección de errores

Varias veces el código generado se probó contra los datos reales y se corrigieron errores que no
eran evidentes solo leyendo el código:

- **Distancia geoespacial:** la primera versión calculaba clustering con distancia euclidiana
  directa sobre grados de latitud/longitud, lo cual es incorrecto. Se recalculó con Haversine y
  la conclusión cambió de "sin clustering" a "clúster confirmado entre los nodos 1, 5 y 14, con
  el nodo 10 como outlier".
- **Comparación agregada de distancias:** incluso con Haversine, el promedio del grupo completo
  seguía sin ser útil porque un solo punto alejado (nodo 10) inflaba el promedio. Se eliminó y se
  dejó únicamente la matriz de distancias por pares.
- **Atribución incorrecta al diccionario de datos:** una versión del notebook afirmaba que el
  diccionario "esperaba" cierto comportamiento de estacionariedad para `Ener_1-3`, algo que el
  documento fuente no dice. Se corrigió el texto.
- **Lectura visual engañosa por escala:** al graficar las medias móviles de las 6 series no
  estacionarias en un mismo eje, las de menor magnitud se veían planas por diferencia de escala,
  no por tener menos tendencia real. Se corrigió con subplots de escala independiente.
- **Métrica no decisiva presentada como evidencia:** el ratio tendencia/ruido se probó contra los
  datos reales y resultó similar en las 6 series; se corrigió el texto para no presentarlo como
  evidencia decisiva.
- **Resultado de Betweenness en cero sin explicar:** antes de reportar que Betweenness daba 0 en
  todos los nodos, se investigó la causa estructural (grafo bipartito sin intermediarios) y se
  verificó con un segundo método, en vez de asumir un error de cálculo o aceptarlo sin más.
- **Señal de causalidad sin verificar:** la significancia puntual en los lags 4-5 de Granger no
  se aceptó como causalidad real hasta chequearla con selección de orden VAR, que la descartó
  como falso positivo.
- **Comparación de coeficientes AR mal construida:** el MAE global de la Fase 2 promediaba la
  constante del modelo con los coeficientes de rezago, en escalas distintas — se corrigió para
  comparar cada coeficiente por separado.
- **Portada del informe técnico desalineada:** la primera versión centraba el bloque de título
  pero dejaba los nombres y la fecha alineados a la izquierda; se corrigió para que todo quedara
  centrado.

Esta verificación —contrastar cada resultado generado contra los datos reales o la visualización
correspondiente— fue un paso manual del equipo en cada entrega, **no una garantía automática de
la IA**.

## Formato general

- *"Ajusta la estructura del repositorio a nuestra plantilla de entrega."*
- *"Genera el README y la declaración de uso de IA con el mismo formato que usamos en el
  Challenge 02."*