Declaración de uso de Inteligencia Artificial
Challenge 03 — Inteligencia Geo-Temporal y de Redes (TechLogistics S.A.)
Integrante: Camilo (Fase 1 — Data Understanding y Geo-Visualización)
Herramienta usada: Claude (Anthropic), como apoyo durante la construcción del notebook de la Fase 1 y la definición de la estructura del repositorio.

Resumen
Usé IA generativa como apoyo en dos frentes: (1) definir la estructura del repositorio siguiendo la plantilla usada en el Challenge 02 del curso, y (2) construir el notebook de la Fase 1 (exploración geoespacial y estacionariedad) contrastando cada resultado contra los CSV reales del proyecto. En más de un punto, correr el código generado contra los datos reales o simplemente mirar la visualización resultante sacó a la luz errores de razonamiento que no eran evidentes leyendo el código a simple vista (detallados en la sección "Auditoría y corrección de errores" más abajo) — esa verificación fue mía, no algo que la IA hiciera de forma autónoma.

Contexto que se le dio a la IA
Se compartieron el enunciado del Challenge 03, el checklist de entrega, el diccionario de datos oficial (con la descripción de cada variable Agro_* y Ener_*, incluyendo qué series se esperaban estacionarias y cuáles no), y los 4 CSV reales del proyecto (agro_clean.csv, agro_noise.csv, ener_clean.csv, ener_noise.csv), para que el código y las conclusiones se ajustaran a los datos reales y no a supuestos genéricos.

Fase 0 — Estructura del repositorio
"Creemos la estructura de carpetas del repo, guíate de la que utilizamos en otros chats pero aplicada a este caso."
"Ahora el informe va en docs" — se corrigió el árbol para eliminar la carpeta informe/ separada y unificar el PDF dentro de docs/.

Fase 1, Tarea 1 — Exploración Geo-Temporal (notebooks/challenge03_fase1.ipynb)
"Empecemos con el notebook con la Fase 1."
"Según esta imagen yo sí diría que hay una concentración de sensores con NDVI bajo que corresponden al 14, 1, 5 y 9. La única excepción sería el 10" — esto llevó a detectar que el cálculo inicial de distancia (euclidiana sobre grados de latitud/longitud) era metodológicamente incorrecto, y a pedir su corrección.
"Distancia promedio entre sensores de NDVI bajo: 777.3 m / Distancia promedio entre todos los sensores: 722.2 m → No hay evidencia de clustering agregado sobre el grupo completo. Debes quitar esto, esa comparación no tiene sentido" — se eliminó la comparación agregada por no ser una métrica útil con solo 4 puntos y un outlier dentro del grupo.

Fase 1, Tarea 2 — Estacionariedad y Windowing
"¿En el diccionario dice explícitamente que Ener 1 a 3 no son estacionarias?" — se revisó el texto original del diccionario y se confirmó que solo menciona correlación alta entre esas variables, sin pronunciarse sobre estacionariedad; se corrigió el notebook para no atribuirle al diccionario una expectativa que no establece.
"Es que la estás comparando en esta tabla" — al ver la tabla ADF/KPSS completa, se identificó que la columna Coinciden ya mostraba el hallazgo relevante (Ener_1, 2, 3 con ADF y KPSS contradictorios) y se reescribió la conclusión para apoyarse en esa evidencia directa, no en una referencia externa.
"Explícame qué es KPSS/ADF, por qué se comparan y qué estamos haciendo acá."
"Según esta gráfica veo más 1, 2, 3 apuntando a estacionario, ¿qué opinas, cómo lo decidimos?" — se identificó que la gráfica compartía escala entre series de magnitudes muy distintas (Ener_6 en cientos vs. Ener_1-3 en decenas), lo que hacía parecer "planas" a las series pequeñas por escala, no por ausencia de tendencia. Se pidió separar en subplots independientes y calcular una métrica cuantitativa (ratio tendencia/ruido) en vez de decidir por lectura visual.
"Entonces márcalo en el notebook como casos ambiguos y di que los trataremos como no estacionarias."
"Cambia el notebook para que no diga [la métrica de fuerza de tendencia como evidencia] y elimina [el párrafo sobre el ratio]" — al ejecutar la métrica contra los datos reales, el ratio resultó similar en las 6 series (no separaba con claridad los casos), así que se descartó como criterio de decisión y se corrigió el texto que la presentaba como evidencia decisiva.

Auditoría y corrección de errores
Varias veces el código o el razonamiento generado se contrastaron contra los datos reales o la visualización resultante, y se corrigieron errores que no eran evidentes solo leyendo el código:

Distancia geoespacial: la primera versión calculaba clustering con distancia euclidiana directa sobre grados de latitud/longitud, lo cual es incorrecto (un grado de longitud no equivale a la misma distancia real que uno de latitud a esta latitud). Comparar contra la lectura visual del mapa reveló el error; se recalculó con la fórmula de Haversine (distancia real en metros) y la conclusión cambió de "sin clustering" a "clúster confirmado entre los nodos 1, 5 y 14, con el nodo 10 como outlier".
Comparación agregada de distancias: incluso después de corregir a Haversine, el promedio agregado del grupo completo seguía sin ser útil porque un solo punto alejado (nodo 10) inflaba el promedio y ocultaba el clúster real de los otros tres. Se eliminó esa comparación y se dejó únicamente la matriz de distancias por pares, que sí es interpretable.
Atribución incorrecta al diccionario de datos: una versión del notebook afirmaba que el diccionario "esperaba" cierto comportamiento de estacionariedad para Ener_1-3. Al revisar el texto original se confirmó que el diccionario no dice eso; se corrigió para no inventar una expectativa que el documento fuente no establece.
Lectura visual engañosa por escala: al graficar las medias móviles de las 6 series no estacionarias en un mismo eje, las series de menor magnitud (Ener_1-3) se veían planas simplemente por la diferencia de escala frente a Ener_6/Ener_7, no por tener menos tendencia real. Se corrigió usando subplots con escala independiente por variable.
Métrica no decisiva presentada como evidencia: el ratio "tendencia/ruido" calculado para desambiguar los casos de ADF/KPSS contradictorio se probó contra los datos reales y resultó similar en las 6 series (no separaba nada); se corrigió el texto para no presentarlo como parte de la evidencia que sustenta la clasificación final.

Esta verificación —contrastar cada resultado generado contra los datos reales, el diccionario de datos, o la visualización correspondiente— fue un paso manual mío en cada entrega, no una garantía automática de la IA.

Formato general
"Guíate de la estructura que utilizamos en otros chats pero aplicada a este caso" (repositorio y README).
