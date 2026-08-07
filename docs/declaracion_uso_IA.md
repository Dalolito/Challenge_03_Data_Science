Declaración de uso de Inteligencia Artificial
Challenge 03 — Inteligencia Geo-Temporal y de Redes (TechLogistics S.A.)
Equipo: Camilo, Samuel Gutiérrez Jaramillo, David Lopera Londoño, Juan Diego Acuña Giraldo
Herramienta usada: Claude (Anthropic) para la Fase 1, Fase 3, Fase 4, la reestructuración final
del notebook y el Informe Técnico; opencode para la Fase 2.

Resumen
Usamos IA generativa como apoyo en cinco frentes: (1) definir la estructura del repositorio, (2)
construir el notebook de la Fase 1 (exploración geoespacial y estacionariedad), (3) las Fases 2, 3
y 4 (procesamiento de señales, grafos, y modelado de decisión), (4) reescribir el notebook completo
con un tono más orientado a negocio y una estructura de encabezados consistente, y (5) construir el
Informe Técnico en PDF a partir de los resultados ya generados por el pipeline. En varios puntos,
correr el código generado contra los datos reales o simplemente mirar la visualización resultante
sacó a la luz errores que no eran evidentes leyendo el código a simple vista (detallados en la
sección "Auditoría y corrección de errores" más abajo) — esa verificación fue nuestra, no algo que
la IA hiciera de forma autónoma.

Contexto que se le dio a la IA
Se compartieron el enunciado del Challenge 03, el checklist de entrega, el diccionario de datos
oficial (con la descripción de cada variable Agro_* y Ener_*, incluyendo qué series se esperaban
estacionarias y cuáles no), y los 4 CSV reales del proyecto (agro_clean.csv, agro_noise.csv,
ener_clean.csv, ener_noise.csv), para que el código y las conclusiones se ajustaran a los datos
reales y no a supuestos genéricos.

Fase 0 — Estructura del repositorio
"Creemos la estructura de carpetas del repo, guíate de la que utilizamos en otros chats pero
aplicada a este caso."
"Ahora el informe va en docs" — se corrigió el árbol para unificar el PDF dentro de docs/.
"Ten en cuenta que quité la carpeta src y assets porque no las estábamos utilizando" — se
actualizó el árbol del repositorio para reflejar la estructura real final.

Fase 1 — Data Understanding y Geo-Visualización (Camilo)
"Empecemos con el notebook con la Fase 1."
"Según esta imagen yo sí diría que hay una concentración de sensores con NDVI bajo que
corresponden al 14, 1, 5 y 9. La única excepción sería el 10" — esto llevó a corregir el cálculo
de distancia (euclidiana sobre grados de latitud/longitud) a la fórmula de Haversine.
"Esa comparación no tiene sentido, debes quitarla" — se eliminó el promedio agregado de
distancias por no ser útil con un outlier dentro del grupo.
"¿En el diccionario dice explícitamente que Ener 1 a 3 no son estacionarias?" — se corrigió el
notebook para no atribuirle al diccionario una expectativa que no establece.
"Es que la estás comparando en esta tabla" — se reescribió la conclusión de estacionariedad para
apoyarse en la columna Coinciden de ADF/KPSS, no en una referencia externa.
"Según esta gráfica veo más 1, 2, 3 apuntando a estacionario, ¿cómo lo decidimos?" — se detectó
que la escala compartida entre series de magnitudes distintas era engañosa; se separó en
subplots independientes.
"Márcalo en el notebook como casos ambiguos y di que los trataremos como no estacionarias."
"Cambia el notebook para que no diga [la métrica de fuerza de tendencia] y elimina [el párrafo
del ratio]" — el ratio tendencia/ruido no separaba las series, se descartó como criterio.

Fase 2 — Procesamiento de Señales y Filtrado (Samuel)
"Ayúdame a plantear el enfoque de la fase: análisis espectral con FFT/PSD y espectrogramas para
Ener_4, y filtrado pasa-bajo con Butterworth para Agro_3, contrastado contra el checklist."
"Redacta las celdas Markdown de las Tareas 3 y 4 con la documentación del análisis."
"Revisa la comparación de coeficientes AR, el MAE global que calculé mezcla la constante con los
lags en escalas distintas, esa conclusión no es válida" — se corrigió la interpretación para
comparar cada coeficiente por separado.
"Dame el código de FFT/PSD (Welch), espectrograma STFT, filtro Butterworth con filtfilt, RMSE de
reconstrucción y pronóstico AR one-step-ahead."

Fase 3 — Análisis de Grafos y Topología de Red (David)
"Dame el código para construir un grafo dirigido con Source_Node y Target_Node, ponderado por
volumen de lecturas de telemetría."
"Betweenness Centrality me está dando 0 en todos los nodos, ayúdame a entender si es un error o
un resultado real antes de reportarlo" — se investigó la estructura del grafo y se confirmó que
era bipartito de 2 capas sin intermediarios, no un error de cálculo.
"Ayúdame a verificar este resultado con otro método antes de darlo por definitivo" — se calculó
Betweenness también sobre la versión no dirigida del grafo, con el submódulo específico de
NetworkX para redes bipartitas, para confirmar el hallazgo con un segundo método.
"Como ni Betweenness ni la Degree Centrality sin ponderar distinguen nodos en AGRO, ayúdame a
usar el volumen ponderado de telemetría para identificar el cuello de botella, tal como lo pide
el enunciado del reto."

Fase 4, P1 y P2 (Juan Diego)
"Ejecuta Causalidad de Granger entre Ener_10 y Ener_9 en ambas direcciones."
"Da significativo en los lags 4 y 5 pero no antes, ¿eso ya es evidencia de causalidad o debería
verificarlo de otra forma?" — se corrió una selección de orden de un modelo VAR (AIC/BIC) como
chequeo independiente, que no encontró estructura autorregresiva real, y se documentó la señal
inicial como falso positivo por comparaciones múltiples.
"El dataset no mide pendiente directamente, ayúdame a aproximarla con la varianza del viento
(Agro_10) y a dejar claro que es un supuesto, no un hallazgo confirmado."
"La correlación entre NDVI y la varianza del viento me dio débil (r≈0.05), no fuerces la
narrativa de que la pendiente explica el NDVI bajo, reporta el resultado tal como salió."

Reestructuración del notebook completo y P3 (Camilo)
"Revisa el notebook de mis compañeros antes de ayudarme a completar el P3."
"Ayúdame a completar el P3 (ARIMAX de la Demanda) en base a lo que hicieron mis compañeros" — se
reutilizó la Degree Centrality de la Fase 3 y la clasificación de estacionariedad de la Fase 1.
"¿Esto tiene sentido?" (sobre el resultado de que la centralidad no mejora el AIC) — se verificó
que AIC, BIC, significancia del coeficiente y correlación cruda apuntaran en la misma dirección
antes de aceptar la conclusión.
"Veo que las secciones de mi compañero no siguen la oratoria que planteamos en la Fase 1, menos
técnica y más de negocio explicando los resultados. Corrige eso en el resto del notebook, y
define una estructura más clara con un título central y las fases divididas claramente" — se
reescribieron las celdas Markdown de las Fases 2, 3, 4 y P3 sin tocar el código, y se agregó un
título central con índice y jerarquía de encabezados consistente.
"¿Estas 4 preguntas de validación del checklist se respondieron?" — se revisó el notebook
completo y se confirmó que ninguna tenía respuesta explícita, lo que llevó a incluirlas en el
Informe Técnico.

Informe Técnico (PDF)
"Ayúdame a hacer el informe técnico, guíate de este que hicimos para el Challenge 2."
"La portada no quedó centrada, quita a Camilo y la fecha es 7/08/2026" — se corrigió la
alineación de la portada y los datos.

Auditoría y corrección de errores
Varias veces el código o el razonamiento generado se contrastaron contra los datos reales o la
visualización resultante, y se corrigieron errores que no eran evidentes solo leyendo el código:

Distancia geoespacial: la primera versión calculaba clustering con distancia euclidiana directa
sobre grados de latitud/longitud, lo cual es incorrecto. Se recalculó con Haversine y la
conclusión cambió de "sin clustering" a "clúster confirmado entre los nodos 1, 5 y 14, con el
nodo 10 como outlier".
Comparación agregada de distancias: incluso con Haversine, el promedio del grupo completo
seguía sin ser útil porque un solo punto alejado (nodo 10) inflaba el promedio. Se eliminó y se
dejó únicamente la matriz de distancias por pares.
Atribución incorrecta al diccionario de datos: una versión del notebook afirmaba que el
diccionario "esperaba" cierto comportamiento de estacionariedad para Ener_1-3, algo que el
documento fuente no dice. Se corrigió el texto.
Lectura visual engañosa por escala: al graficar las medias móviles de las 6 series no
estacionarias en un mismo eje, las de menor magnitud se veían planas por diferencia de escala,
no por tener menos tendencia real. Se corrigió con subplots de escala independiente.
Métrica no decisiva presentada como evidencia: el ratio tendencia/ruido se probó contra los
datos reales y resultó similar en las 6 series; se corrigió el texto para no presentarlo como
evidencia decisiva.
Resultado de Betweenness en cero sin explicar: antes de reportar que Betweenness daba 0 en
todos los nodos, se investigó la causa estructural (grafo bipartito sin intermediarios) y se
verificó con un segundo método, en vez de asumir un error de cálculo o aceptarlo sin más.
Señal de causalidad sin verificar: la significancia puntual en los lags 4-5 de Granger no se
aceptó como causalidad real hasta chequearla con selección de orden VAR, que la descartó como
falso positivo.
Portada del informe técnico desalineada: la primera versión centraba el bloque de título pero
dejaba los nombres y la fecha alineados a la izquierda; se corrigió para que todo quedara
centrado.

Esta verificación —contrastar cada resultado generado contra los datos reales, el diccionario de
datos, o la visualización correspondiente— fue un paso manual del equipo en cada entrega, no una
garantía automática de la IA.

Formato general
"Guíate de la estructura que utilizamos en otros chats pero aplicada a este caso" (repositorio y
README).
"Dame el README y la declaración de uso de IA únicamente de lo que llevamos, para que mis
compañeros sigan el mismo formato."
"En la declaración no dejes zonas incompletas o por llenar, créalas según lo que ves en el
notebook y las que ya están."