# Segmentación de entidades federativas de México según patrones de accidentes de tránsito: el caso de Morelos

**Autores:** Ruiz Santos Orlando, Ventura Gil Astrid Valeria, Ocampo Martínez Angel Daniel, Medina Villa Abril Aidee
**Materia:** Extracción de Conocimiento de Bases de Datos — 9° A
**Palabras clave:** accidentes de tránsito, movilidad, datos abiertos, aprendizaje no supervisado, KMeans, Morelos

## Resumen

Este artículo analiza datos públicos oficiales de accidentes de tránsito en México, publicados por la SICT en datos.gob.mx. A partir de dos conjuntos de datos —accidentes por día/condición de luz, y por mes con heridos y muertos— se construyeron variables de tasa y proporción por entidad federativa (letalidad, tasa de heridos, proporción en fin de semana y en horario nocturno/crepuscular). Con ellas se aplicó KMeans para agrupar a las 32 entidades y ubicar en qué grupo cae Morelos y qué lo caracteriza. El análisis sugiere que Morelos no destaca por la severidad de sus accidentes —su letalidad está entre las más bajas del país— sino por una concentración temporal inusual: la proporción más alta de accidentes nocturnos/crepusculares del país y la segunda más alta en fin de semana. Se discuten las limitaciones: agregación estatal, muestra pequeña e imposibilidad de derivar causalidad.

## Introducción

Los accidentes de tránsito suelen analizarse por conteo bruto de casos, más que como un problema de *patrones*. Preguntas como "¿en qué se parece Morelos a otros estados en accidentalidad?" requieren comparar patrones, no solo volúmenes.

Este proyecto integra el ciclo completo de un análisis de datos aplicado —búsqueda de fuente pública, limpieza, análisis exploratorio, modelado no supervisado, interpretación y limitaciones— en el marco del **Tema 4: movilidad, transporte o accidentes con datos públicos**, con foco en Morelos, sede de la institución de los autores.

La pregunta guía es: *¿qué patrones aparecen en los datos de accidentes de tránsito en México, y en qué grupo se ubica Morelos frente a las demás entidades?* No es un estudio causal, sino un análisis descriptivo por semejanza estadística en un conjunto acotado de variables.

## Fuente de datos

Los datos provienen del dataset **"Accidentes de tránsito"**, publicado por la SICT en el Portal de Datos Abiertos de México (`datos.gob.mx/dataset/accidentes_transito`), bajo licencia CC-BY-4.0. Se usaron dos recursos: **por día de la semana** (`sct_70_accidentes_dia.csv`, totales y mortales por día y por condición de luz) y **por mes** (`sct_71_accidentes_mes.csv`, accidentes, heridos, muertos y daños materiales por entidad y mes).

Del recurso mensual se utilizaron los conteos de accidentes, heridos y muertos para construir las tasas de severidad; la variable de daños materiales estimados (en millones de pesos) se revisó durante la exploración pero se excluyó, ya que introduce una dimensión económica ajena al comportamiento vial que se busca describir.

Ambos están agregados a nivel de **entidad federativa**, no municipal. Se buscó una fuente con desagregación municipal para Morelos (incluido el programa ATUS de INEGI), pero no se logró un CSV de descarga directa con ese nivel en el tiempo disponible (búsqueda documentada en `referencias/fuentes_datos.txt`), por lo que el análisis se hace a nivel nacional, usando a Morelos como caso de estudio. Los metadatos del recurso, consultados vía la API de CKAN de datos.gob.mx, solo exponen las fechas de creación y actualización en el portal (julio–agosto de 2025), sin especificar el año exacto de los registros; esto se documenta como limitación.

## Preparación de datos

No se encontraron nulos ni filas duplicadas, y ambos archivos cubrían las 32 entidades. Se hicieron dos ajustes: en el archivo por día, la columna `accidentes` no es un conteo sino una **etiqueta de tipo de registro** (cada entidad aparece dos veces: `"accidentes"` y `"accidentes mortales"`), por lo que fue necesario separarlos antes de construir variables; y los nombres de entidades en formato no estándar (p. ej. `"Ciudad De Mexico"`) se homologaron a la forma estándar para combinar ambos archivos sin errores.

Ambos datasets se combinaron por entidad mediante una unión interna (`inner join`) por nombre; del mensual se calcularon totales anuales de accidentes, heridos y muertos, y del diario, la fracción de accidentes en fin de semana y en condición de luz crepuscular/nocturna. Tras la homologación de nombres, las 32 entidades aparecían en ambos archivos, por lo que no se perdió ninguna fila.

Se optó por **tasas y proporciones**, no solo totales: comparar directamente los accidentes de la Ciudad de México con los de Colima favorecería a los estados grandes por tamaño de población y parque vehicular. Normalizar por el total de accidentes hace la comparación más justa y orientada a *patrones*. El dataset limpio, con 32 filas, se guardó en `data/processed/dataset_limpio.csv`.

## Análisis exploratorio

La tasa de letalidad nacional promedia **~24.6 muertos por cada 100 accidentes**, con los valores más altos en Coahuila (40.4) y Jalisco (38.0), y los más bajos en Tlaxcala (6.7), Tabasco (10.3) y Morelos (11.4) —los tres únicos estados por debajo de 12—, lo cual ya adelanta que Morelos no es un caso típico en severidad. La proporción de fin de semana promedia **36.6%**, y la nocturna/crepuscular **46.1%**.

Morelos: en volumen total ocupa el lugar 24 de 32 (201 accidentes, 118 heridos, 23 fallecidos); en letalidad registra **~11.4**, de los tres valores más bajos del país (lugar 30 de 32); en fin de semana, **47.8%**, la segunda más alta tras la Ciudad de México (49.3%); y en horario nocturno/crepuscular, **54.7%**, la más alta de las 32 entidades, seguida de cerca por Tlaxcala (53.4%). Es notable que estas dos entidades —Morelos y Tlaxcala— comparten tanto una letalidad baja como una alta concentración nocturna, lo que sugiere que este patrón temporal no es exclusivo de Morelos, sino potencialmente compartido por entidades del centro del país con características similares de movilidad.

Esta combinación —letalidad baja pero concentración temporal muy por encima del promedio nacional— motiva un modelo que combine varias variables a la vez, en lugar de analizar cada una por separado. Se ilustra en la Gráfica 1, con Morelos resaltado en rojo dentro del volumen total de accidentes por entidad; el patrón temporal por cluster se retoma más adelante en la Gráfica 2, una vez presentado el modelo.

![Gráfica 1. Accidentes totales por entidad federativa, Morelos resaltado en rojo.](imagenes/grafica_1.png)

**Gráfica 1.** Accidentes totales por entidad federativa, Morelos resaltado en rojo. Se observa que el volumen no distingue a Morelos de forma particular: aparece en una posición intermedia-baja, consistente con su tamaño poblacional.

## Metodología

Se aplicó **KMeans** porque el objetivo no es predecir una etiqueta ya conocida, sino **descubrir agrupaciones** con patrones similares; no existe una variable objetivo previa (como "tipo de entidad") que justifique un enfoque supervisado. Se prefirió sobre alternativas como el agrupamiento jerárquico por producir un número fijo y fácil de comunicar de grupos, conveniente dado que el objetivo final es explicar la posición de una sola entidad (Morelos) frente al resto.

Se usaron cuatro variables (letalidad, heridos, proporción de fin de semana y proporción nocturna/crepuscular), sin variables de volumen, para que el agrupamiento reflejara *comportamiento* y no separara simplemente estados grandes de pequeños. Se estandarizaron con `StandardScaler`, ya que tienen escalas distintas y, sin este paso, la de mayor escala dominaría el cálculo de distancias.

Para elegir *k* se calculó el coeficiente de silueta entre 2 y 6 (valores entre 0.246 y 0.299, sin separación fuertemente marcada, algo esperable con solo 32 observaciones). Se optó por **k=3**, con coeficiente competitivo y un número de grupos manejable e interpretable. El modelo se ajustó con semilla fija (`random_state=42`) y diez inicializaciones (`n_init=10`) para reducir la sensibilidad de KMeans a la inicialización aleatoria de los centroides y hacer el resultado reproducible entre ejecuciones.

## Resultados

El modelo con k=3 agrupó a las 32 entidades en clusters de 15, 11 y 6. Morelos quedó en el **cluster 0** (15 entidades), junto con Jalisco, Guerrero, Puebla, Guanajuato y Estado de México, entre otros (lista completa en `outputs/resultados_modelo.csv`).

| Cluster | n | Letalidad (prom.) | Heridos (prom.) | % fin de semana | % nocturno |
|---|---|---|---|---|---|
| 0 (incluye Morelos) | 15 | 27.8 | 73.0 | 38.6% | 48.1% |
| 1 | 11 | 16.5 | 65.6 | 33.1% | 46.5% |
| 2 | 6 | 31.1 | 111.0 | 38.3% | 40.3% |

Los tres clusters tienen perfiles distinguibles: el **cluster 1** (11 entidades, entre ellas Chiapas, Querétaro y Veracruz) tiene la letalidad promedio más baja; el **cluster 2** (el más pequeño, 6 entidades, incluida la Ciudad de México y Coahuila) tiene la tasa de heridos más alta y la proporción nocturna más baja, un patrón más diurno; y el **cluster 0**, donde cae Morelos, combina letalidad relativamente alta con la mayor concentración de fin de semana y horario nocturno de los tres.

Morelos resulta un valor atípico dentro de su propio cluster: en letalidad (11.4) está muy por debajo del promedio del grupo (27.8), pero en patrón temporal se ubica en el extremo superior de las 32 entidades, como muestra la Gráfica 2.

![Gráfica 2. Proporción de accidentes en fin de semana vs. horario nocturno/crepuscular, coloreada por cluster; Morelos marcado con una estrella.](imagenes/grafica_2.png)

**Gráfica 2.** Proporción de accidentes en fin de semana (eje horizontal) contra proporción en horario nocturno/crepuscular (eje vertical), coloreada por cluster de KMeans. Morelos (estrella negra) aparece desplazado hacia la esquina superior derecha, separado del centro de su propio cluster (azul), lo que confirma visualmente que su patrón temporal es más extremo que el del resto de las entidades agrupadas con él.

## Interpretación

Con los datos disponibles se observa que Morelos queda agrupado con estados de letalidad relativamente alta **no porque comparta ese nivel de letalidad**, sino porque comparte con ellos un patrón temporal marcado: en conjunto, sus cuatro variables resultaron más cercanas a ese grupo que a los otros dos.

Esto sugiere una posible asociación entre los accidentes en Morelos y patrones de movilidad nocturna y de fin de semana. Sin embargo, **este análisis no permite afirmar causalidad**: no se cuenta con datos sobre causas, tipo de vía, tipo de vehículo o consumo de alcohol, necesarios para eso. Sí se puede decir con confianza que el patrón temporal de Morelos es estadísticamente distinto —y más marcado— que el de la mayoría de las demás entidades, lo cual ilustra un punto metodológico: un modelo de agrupamiento puede colocar una observación en un grupo por razones distintas a las esperadas.

## Limitaciones

- **Nivel de agregación:** datos por entidad (32 registros), no municipal; no fue posible replicar el análisis por municipio de Morelos con fuentes descargables en el plazo disponible.
- **Periodo no explicitado:** los metadatos solo indican fechas de publicación/actualización (2025), no el año exacto de referencia.
- **Tamaño de muestra:** 32 observaciones limitan la robustez del agrupamiento; el mejor coeficiente de silueta (0.299) es moderado.
- **Variables faltantes:** no se contó con tipo de vehículo, tipo de vialidad ni causa reportada.
- **Ausencia de causalidad:** el modelo agrupa por semejanza estadística, no demuestra que el horario nocturno o el fin de semana *causen* más accidentes.
- **Dependencia de las variables elegidas:** otro conjunto de variables probablemente produciría una segmentación distinta.

## Conclusiones

Este proyecto aplicó un flujo completo de análisis de datos públicos —búsqueda de fuente oficial, limpieza, construcción de variables, análisis exploratorio, modelado no supervisado e interpretación— centrado en el patrón de accidentalidad de Morelos frente a las demás entidades.

Con los datos disponibles, Morelos no se distingue por la severidad de sus accidentes —su letalidad está entre las más bajas del país— sino por una concentración temporal particular: la proporción más alta de accidentes nocturnos/crepusculares del país y la segunda más alta en fin de semana. Esto no debe leerse como explicación definitiva, sino como punto de partida para preguntas de seguimiento: con datos más desagregados (municipales, por vialidad o causa) sería posible explorar si el patrón se relaciona con zonas, vehículos o comportamientos de movilidad específicos.

Una continuación natural de este trabajo sería gestionar acceso a microdatos municipales para Morelos e incorporar variables de causa y tipo de vialidad, para pasar de una asociación descriptiva a hipótesis más específicas.

## Referencias

1. SICT. *Accidentes de tránsito* [conjunto de datos]. datos.gob.mx: https://www.datos.gob.mx/dataset/accidentes_transito (consultado el 22 de agosto de 2026). Licencia CC-BY-4.0.
2. INEGI. *Accidentes de Tránsito Terrestre en Zonas Urbanas y Suburbanas (ATUS)*: https://www.inegi.org.mx/programas/accidentes/ (fuente exploratoria, ver `referencias/fuentes_datos.txt`).
3. Pedregosa, F. et al. (2011). *Scikit-learn: Machine Learning in Python*. JMLR, 12, 2825-2830.
4. McKinney, W. (2010). *Data Structures for Statistical Computing in Python*. Proc. 9th Python in Science Conf.