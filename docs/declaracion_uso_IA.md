Declaración de uso de Inteligencia Artificial
Challenge 03 — Inteligencia Geo-Temporal y de Redes (TechLogistics S.A.)
Equipo: Camilo (Fase 1 y cierre del proyecto), Samuel (Fase 2), David (Fase 3), Juan Diego (Fase 4)
Herramienta usada: Claude (Anthropic) para la Fase 1, Fase 3, Fase 4, la reestructuración final del
notebook y el Informe Técnico; opencode para la Fase 2.

Resumen
Usamos IA generativa como apoyo en cinco frentes: (1) definir la estructura del repositorio
siguiendo la plantilla usada en el Challenge 02 del curso, (2) construir el notebook de la Fase 1
(exploración geoespacial y estacionariedad), (3) las Fases 2, 3 y 4 (procesamiento de señales,
grafos, y modelado de decisión), (4) reescribir el notebook completo con un tono más orientado a
negocio y una estructura de encabezados consistente entre fases, y (5) construir el Informe
Técnico en PDF a partir de los resultados ya generados por el pipeline. En más de un punto, correr
el código generado contra los datos reales o simplemente mirar la visualización resultante sacó a
la luz errores de razonamiento que no eran evidentes leyendo el código a simple vista (detallados
en la sección "Auditoría y corrección de errores" de cada fase) — esa verificación fue nuestra, no
algo que la IA hiciera de forma autónoma.

Contexto que se le dio a la IA
Se compartieron el enunciado del Challenge 03, el checklist de entrega, el diccionario de datos
oficial (con la descripción de cada variable Agro_* y Ener_*, incluyendo qué series se esperaban
estacionarias y cuáles no), y los 4 CSV reales del proyecto (agro_clean.csv, agro_noise.csv,
ener_clean.csv, ener_noise.csv), para que el código y las conclusiones se ajustaran a los datos
reales y no a supuestos genéricos.

Fase 0 — Estructura del repositorio
"Creemos la estructura de carpetas del repo, guíate de la que utilizamos en otros chats pero
aplicada a este caso."
"Ahora el informe va en docs" — se corrigió el árbol para eliminar la carpeta informe/ separada y
unificar el PDF dentro de docs/.
"Ahora ayudame a terminar el README y la declaración de uso de IA, ten en cuenta que quité la
carpeta src y assets porque no las estábamos utilizando" — se actualizó el árbol del repositorio
para reflejar la estructura real final, sin los módulos de Python ni las capturas de pantalla que
se habían planeado pero nunca se usaron.

Fase 1, Tarea 1 — Exploración Geo-Temporal
"Empecemos con el notebook con la Fase 1."
"Según esta imagen yo sí diría que hay una concentración de sensores con NDVI bajo que
corresponden al 14, 1, 5 y 9. La única excepción sería el 10" — esto llevó a detectar que el
cálculo inicial de distancia (euclidiana sobre grados de latitud/longitud) era metodológicamente
incorrecto, y a pedir su corrección.
"Distancia promedio entre sensores de NDVI bajo: 777.3 m / Distancia promedio entre todos los
sensores: 722.2 m → No hay evidencia de clustering agregado sobre el grupo completo. Debes
quitar esto, esa comparación no tiene sentido" — se eliminó la comparación agregada por no ser
una métrica útil con solo 4 puntos y un outlier dentro del grupo.

Fase 1, Tarea 2 — Estacionariedad y Windowing
"¿En el diccionario dice explícitamente que Ener 1 a 3 no son estacionarias?" — se revisó el texto
original del diccionario y se confirmó que solo menciona correlación alta entre esas variables,
sin pronunciarse sobre estacionariedad; se corrigió el notebook para no atribuirle al diccionario
una expectativa que no establece.
"Es que la estás comparando en esta tabla" — al ver la tabla ADF/KPSS completa, se identificó que
la columna Coinciden ya mostraba el hallazgo relevante (Ener_1, 2, 3 con ADF y KPSS
contradictorios) y se reescribió la conclusión para apoyarse en esa evidencia directa.
"Según esta gráfica veo más 1, 2, 3 apuntando a estacionario, ¿qué opinas, cómo lo decidimos?" —
se identificó que la gráfica compartía escala entre series de magnitudes muy distintas, lo que
hacía parecer "planas" a las series pequeñas por escala, no por ausencia de tendencia. Se separó
en subplots independientes y se calculó una métrica cuantitativa (ratio tendencia/ruido).
"Entonces márcalo en el notebook como casos ambiguos y di que los trataremos como no
estacionarias."
"Cambia el notebook para que no diga [la métrica de fuerza de tendencia como evidencia] y
elimina [el párrafo sobre el ratio]" — al ejecutar la métrica contra los datos reales, el ratio
resultó similar en las 6 series, así que se descartó como criterio de decisión.

Fases 2, 3 y 4 — Revisión del trabajo del equipo y P3 (Analítica Predictiva)
Con el notebook de Fases 1-2 ya construido por Camilo y Samuel, el equipo avanzó las Fases 3
(David) y 4 P1-P2 (Juan Diego) directamente en el notebook compartido. Para cerrar el taller, se le
pidió a la IA:
"Revisa el notebook de mis compañeros antes de ayudarme a completar el P3" — la IA leyó las 109
celdas existentes (Fases 1 a 4, P1 y P2) para entender el estilo, las decisiones ya tomadas
(clasificación de estacionariedad, centralidad de nodos) y no duplicar trabajo ni contradecir
hallazgos previos.
"Ayúdame a completar el P3 (ARIMAX de la Demanda) en base a lo que hicieron mis compañeros" —
se construyó el modelo reutilizando la Degree Centrality ya calculada en la Fase 3 y la
clasificación de estacionariedad de la Fase 1, en vez de recalcular esas piezas desde cero.
"¿Esto tiene sentido?" (sobre el resultado de que la centralidad no mejora el AIC) — se verificó
que el AIC, el BIC, la significancia del coeficiente y la correlación cruda apuntaran todos en la
misma dirección antes de aceptar la conclusión como sólida, y se identificó un matiz (p=0.066 es
zona gris, no un "no" contundente) que se dejó documentado con más precisión.

Reestructuración del notebook completo
"Veo que las secciones de mi compañero no siguen la oratoria que planteamos en la Fase 1, menos
técnica y más de negocio explicando los resultados. Quiero que corrijas eso en el resto del
notebook, además define una estructura más clara con un título central, las fases y los retos
divididos claramente" — se reescribieron las 45 celdas Markdown de las Fases 2, 3, 4 y P3 para que
cada tarea abriera con la pregunta de negocio antes del método, y se separaran explícitamente las
conclusiones en "Para negocio" y "Evidencia técnica". Se agregó una celda de título central con
tabla-índice de las 4 fases, y se unificó la jerarquía de encabezados (Fase → Tarea/Pregunta →
Subsección) en todo el documento. No se modificó ninguna celda de código.

Preguntas de Validación del checklist
"¿Estas preguntas se respondieron?" (las 4 preguntas de Auto-Evaluación del checklist) — se revisó
el notebook completo buscando cada pregunta y se confirmó que ninguna tenía una respuesta
explícita y directa, aunque había evidencia dispersa relevante para varias de ellas. Esto llevó a
construir el Informe Técnico con una sección dedicada a responderlas.

Informe Técnico (PDF)
"Ayúdame a hacer el informe técnico, guíate de este que hicimos para el Challenge 2" — se generó
el PDF con reportlab siguiendo el mismo formato del informe del Challenge 02 (Times-Roman,
sin colores de marca, mínima negrilla), con figuras estáticas regeneradas en matplotlib a partir
del pipeline real (no capturas de los gráficos interactivos de Plotly), y una sección dedicada a
las 4 Preguntas de Validación del checklist, ausente hasta ese momento.
"La portada no quedó centrada, quita a Camilo y la fecha es 7/08/2026" — se corrigió la
alineación de la portada (estaba centrada solo en el bloque de título, no en los nombres y la
fecha) y se ajustaron los datos.

Auditoría y corrección de errores
Varias veces el código o el razonamiento generado se contrastaron contra los datos reales o la
visualización resultante, y se corrigieron errores que no eran evidentes solo leyendo el código:

Distancia geoespacial: la primera versión calculaba clustering con distancia euclidiana directa
sobre grados de latitud/longitud, lo cual es incorrecto. Se recalculó con la fórmula de Haversine
(distancia real en metros) y la conclusión cambió de "sin clustering" a "clúster confirmado entre
los nodos 1, 5 y 14, con el nodo 10 como outlier".
Comparación agregada de distancias: incluso después de corregir a Haversine, el promedio
agregado del grupo completo seguía sin ser útil porque un solo punto alejado (nodo 10) inflaba el
promedio. Se eliminó esa comparación y se dejó únicamente la matriz de distancias por pares.
Atribución incorrecta al diccionario de datos: una versión del notebook afirmaba que el
diccionario "esperaba" cierto comportamiento de estacionariedad para Ener_1-3, algo que el
documento fuente no dice. Se corrigió el texto.
Lectura visual engañosa por escala: al graficar las medias móviles de las 6 series no
estacionarias en un mismo eje, las series de menor magnitud se veían planas por diferencia de
escala, no por tener menos tendencia real. Se corrigió con subplots de escala independiente.
Métrica no decisiva presentada como evidencia: el ratio "tendencia/ruido" se probó contra los
datos reales y resultó similar en las 6 series; se corrigió el texto para no presentarlo como
evidencia decisiva.
Portada del informe técnico desalineada: la primera versión del PDF centraba el bloque de
título pero dejaba los nombres del equipo y la fecha alineados a la izquierda; se corrigió para que
todo el bloque de la portada quedara centrado de forma consistente.

Esta verificación —contrastar cada resultado generado contra los datos reales, el diccionario de
datos, o la visualización correspondiente— fue un paso manual del equipo en cada entrega, no una
garantía automática de la IA.

## Fase 2 — Procesamiento de Señales y Filtrado (Samuel)

En la Fase 2 el apoyo de la IA fue más puntual que en la Fase 1, en cuatro frentes:

- **Planeación:** definí junto con la IA el enfoque de la fase (análisis espectral con FFT/PSD y
  espectrogramas, y filtrado pasa-bajo con Butterworth) contrastado contra el checklist de
  entrega, y el alcance de cada tarea antes de escribir código.
- **Documentación:** la IA redactó las celdas Markdown del notebook y las respuestas a las
  Tareas 3 y 4, que luego ajusté para que quedaran en mi tono y reflejaran mi análisis.
- **Buena escritura:** revisión de redacción y consistencia de las conclusiones (por ejemplo,
  corregir una interpretación de coeficientes que contradecía sus propios números).
- **Algunas cosas de código:** FFT/PSD (Welch), espectrogramas STFT, filtro Butterworth con
  `filtfilt`, RMSE de reconstrucción, pronóstico AR one-step-ahead y comparación de coeficientes
  AR(2) entre las versiones clean, ruidosa y filtrada.

### Decisiones tomadas por mí
Las decisiones técnicas relevantes las tomé yo: por ejemplo, tras ver los trade-offs que la IA me
presentó (distintos cutoff para el Butterworth), elegí el cutoff = 0.15 porque mejoraba el error de
pronóstico manteniendo la varianza de la serie original, aun sabiendo que distorsiona la estimación
de los coeficientes AR(2).

### Auditoría y verificación
- Verifiqué la ejecución completa del notebook contra los datos reales y contra los resultados ya
  validados de la Fase 1 (las salidas de la Fase 1 se restauraron intactas tras la re-ejecución).
- Detecté y corregí, con la IA, una conclusión de la comparación de coeficientes que contradecía
  sus propios datos (el MAE global promediaba `const` con los lags, en escalas distintas).

## Fase 3 — Análisis de Grafos y Topología de Red (David)

El notebook muestra un uso de IA orientado a verificar hallazgos estructurales antes de
aceptarlos, siguiendo el mismo patrón de auditoría que las Fases 1 y 2.

- **Planeación:** el grafo se construyó de forma dirigida usando `Source_Node` y `Target_Node`
  como nodos reales de la red (no las variables de sensor), ponderado por volumen de lecturas de
  telemetría — una decisión explícita para que el grafo capturara tráfico, no solo topología.
- **Verificación antes de aceptar un resultado en cero:** al calcular Betweenness Centrality y
  obtener 0 en todos los nodos de ambas redes, el notebook no se quedó con ese resultado sin
  explicarlo. Se investigó primero la estructura del grafo (chequeo de solapamiento entre
  `Source_Node` y `Target_Node`) para confirmar que la razón era estructural — una red bipartita
  de 2 capas sin intermediarios — y no un error de cálculo. Después se corrió una segunda
  verificación independiente (Betweenness sobre la versión no dirigida del grafo, con el
  submódulo específico `networkx.algorithms.bipartite`) para confirmar el hallazgo con un método
  distinto antes de reportarlo como definitivo.
- **Decisión propia ante una métrica que no servía:** como ni Betweenness ni, en el caso de AGRO,
  la Degree Centrality sin ponderar distinguían nodos (por ser un grafo bipartito completo), se
  decidió usar el volumen ponderado de telemetría como criterio para identificar el nodo cuello
  de botella, interpretando el enunciado del reto de forma literal ("el nodo por donde pasa la
  mayor cantidad de información de telemetría").
- **Auditoría:** el hallazgo del Nodo 119 en la red ENER se sostuvo con tres métricas
  independientes (Degree Centrality, grado ponderado, Betweenness no dirigida) antes de darlo
  por definitivo — el mismo estándar de "no aceptar el primer resultado significativo sin
  verificación cruzada" que se usó en la Fase 1 con ADF/KPSS.

## Fase 4, P1 y P2 (Juan Diego)

**P1 — Causalidad y Redes:** el notebook muestra que una primera corrida de Causalidad de
Granger entre `Ener_10` y `Ener_9` arrojó significancia en los rezagos 4 y 5 (p≈0.02). En vez de
reportar esto como "existe causalidad", se pidió una verificación independiente con selección de
orden de un modelo VAR (criterios AIC/BIC) antes de aceptar el hallazgo — el mismo patrón de
auditoría de las fases anteriores, aplicado aquí a un resultado de p-valor que a primera vista
parecía positivo. Como el AIC/BIC no mostró ninguna estructura autorregresiva cruzada real
(seleccionaron lag=0), se documentó la señal inicial como un falso positivo por comparaciones
múltiples en vez de forzar una conclusión de causalidad que los datos no sostenían de forma
robusta. La parte hipotética de la pregunta (impacto de un fallo en el nodo de mayor Betweenness)
se respondió igual, dejando explícito que la premisa de causalidad no se cumplía con la evidencia
disponible, en vez de omitir esa aclaración.

**P2 — Optimización Geo-Agrónoma:** el notebook sigue un flujo de 4 pasos explícitos (filtrar el
jitter GPS, localizar los sensores de menor NDVI, evaluar el proxy de pendiente, y emitir la
recomendación), con una decisión metodológica propia: como el dataset no mide pendiente
directamente, se decidió aproximarla con la varianza del viento (`Agro_10`), documentando ese
supuesto como una premisa del enunciado y no como un hallazgo confirmado. Al encontrar que la
correlación entre NDVI y esa varianza era débil (r≈0.05), no se forzó la narrativa de que la
pendiente explica el NDVI bajo — se reportó el resultado tal como salió, y se usó únicamente el
hallazgo puntual del nodo 10 (mayor varianza de viento de la red) como evidencia de riesgo,
distinguiendo explícitamente entre lo que los datos confirman y lo que sigue siendo un supuesto
de diseño del taller.

Formato general
"Guíate de la estructura que utilizamos en otros chats pero aplicada a este caso" (repositorio y
README).
"Dame el README y la declaración de uso de IA únicamente de lo que llevamos, para que mis
compañeros sigan el mismo formato."
"En la declaración no dejes zonas incompletas o por llenar, créalas según lo que ves en el
notebook y las que ya están" — las secciones de Fase 3 y Fase 4 (P1, P2) se redactaron a partir
de la evidencia visible en el propio notebook (decisiones metodológicas, verificaciones cruzadas,
y manejo de resultados negativos o ambiguos), siguiendo el mismo patrón de auditoría documentado
en las Fases 1 y 2, en vez de dejarse como plantilla vacía.