# Diccionario de columnas auxiliares de la base procesada

La base común de 2023 a 2025 contiene **4.985.130 egresos y 36 columnas**. Incluye los 16 campos compartidos por los tres CSV, `ERROR` del CSV de 2025 y 19 columnas auxiliares. Cada fila representa un egreso hospitalario. Las columnas auxiliares registran la procedencia, las revisiones de calidad y las descripciones obtenidas de los catálogos.

`ERROR` es un campo entero que admite ausentes. Conserva 1.705.407 ceros y 107 unos en 2025. En 2023 y 2024 queda como `null` porque sus CSV carecen del campo. Su significado está sin documentar. La presencia de un valor distingue a 2025 de los dos años anteriores, por lo que se reserva inicialmente para revisar la calidad de los datos.

El [notebook de preprocesamiento](../01_preprocesamiento_egresos_2025.ipynb) produce la [base común](../datos/procesados/egresos_2023_2025_base_comun.parquet). Las descripciones CIE-10 proceden del mismo diccionario incluido en cada ZIP. La correspondencia territorial está en el [catálogo geográfico](../datos/originales/catalogo_geografico_dpa_2023.csv).

| Categoría | Columnas auxiliares |
| --- | ---: |
| Procedencia, control y calidad de datos | 7 |
| Información geográfica | 4 |
| Información de diagnósticos CIE-10 | 7 |
| Control de estadías extremas | 1 |
| **Total** | **19** |

Los ejemplos ilustran las reglas de cada columna. Los conteos y ejemplos observados de esta guía corresponden a **2025**. Los informes de cada año están en `resultados/egresos_<año>/tablas/`.

`True` significa verdadero y `False`, falso. Un valor ausente puede mostrarse como `<NA>`: no equivale a cero ni a una cadena de texto. En cambio, `SIN_DATO` es una etiqueta explícita de los campos de estado.

## 1. Control y calidad de datos

| Columna | Qué significa y cómo se construye | Ejemplo ilustrativo e interpretación | Para qué sirve y qué precaución considerar |
| --- | --- | --- | --- |
| `ID_REGISTRO` | Número de fila dentro de cada CSV, a partir de 1. | Con `ANO_FUENTE = 2023`, `1` identifica el primer egreso del CSV de 2023. | El par `ANO_FUENTE` e `ID_REGISTRO` permite ubicar la fila original. |
| `ANO_FUENTE` | Año que indica el nombre del ZIP leído. | Una fila de `EGRESOS_2024.zip` tiene `ANO_FUENTE = 2024`. | Identificar el archivo de procedencia incluso cuando `ANO_EGRESO` esté ausente. |
| `TIENE_ASTERISCO` | Indica si alguno de los 16 campos comunes era exactamente `*` antes de la limpieza. | `True` indica que el registro tenía al menos un marcador. | Reconocer la falta de información de origen y comparar su distribución. |
| `TIENE_TEXTO_DANADO` | Busca el carácter de reemplazo `�` en los 16 campos comunes. | `True` puede mantenerse aunque una versión derivada ya tenga el nombre reparado. | Auditar la calidad del texto recibido. |
| `EN_GRUPO_REPETIDO` | Marca las filas que comparten los 16 campos comunes con otra fila del mismo CSV. Se calcula antes de limpiar, con `duplicated(keep=False)`. | Si dos filas coinciden en los 16 campos, ambas reciben `True`. | Examinar repeticiones conservando todos los egresos. |
| `DIAG2_VACIO_ORIGEN` | Indica si `DIAG2` era una cadena vacía (`''`) en el CSV, antes de transformar faltantes. | `True` significa que no se registró un código de causa externa en ese campo. | Distinguir vacíos de origen. `DIAG2` es causa externa, no una segunda enfermedad; un vacío puede ser estructural y no demuestra ausencia de comorbilidades. |
| `TEXTO_DANADO_PENDIENTE` | Busca `�` en las dos columnas geográficas con sufijo `_CORREGIDA`, después del cruce territorial. | `False` significa que esas dos versiones no contienen el carácter de reemplazo. | Comprobar el resultado de la reparación geográfica. No revisa todo el texto de la base ni garantiza que las glosas originales estén reparadas. |

**Observado en las filas con `ANO_FUENTE = 2025`**

| Columna | Resultado observado |
| --- | ---: |
| `ID_REGISTRO` | De 1 a 1.705.514, sin repeticiones |
| `ANO_FUENTE` | 2025 en las 1.705.514 filas |
| `TIENE_ASTERISCO` | 118.961 valores `True` |
| `TIENE_TEXTO_DANADO` | 763.636 valores `True` |
| `EN_GRUPO_REPETIDO` | 660.640 valores `True` |
| `DIAG2_VACIO_ORIGEN` | 1.540.934 valores `True` |
| `TEXTO_DANADO_PENDIENTE` | 0 valores `True` |

**Ejemplo ilustrativo de repetición:** dos hospitalizaciones distintas pueden compartir sexo, edad agrupada, residencia, diagnóstico, días de estadía y los demás atributos públicos. La base no permite comprobar si pertenecen a la misma persona. Por eso `EN_GRUPO_REPETIDO=True` no autoriza a borrar ninguna de ellas ni permite agrupar por paciente.

En 2025 hay **660.640 registros que integran grupos repetidos**. Si se cuenta solo después de la primera aparición, hay **504.278 repeticiones adicionales**.

## 2. Información geográfica

Se conservan `GLOSA_COMUNA_RESIDENCIA` y `GLOSA_REGION_RESIDENCIA`, y se crean versiones corregidas por código. La referencia es la [capa DPA publicada por la SMA](https://ideserver.sma.gob.cl/arcgis/rest/services/IDE/Limites/MapServer/2), con metadatos del grupo interinstitucional de división político-administrativa de 2023. El notebook documenta la consulta del 8 de noviembre de 2026 y la huella de la copia local.

El catálogo consultado tiene 345 comunas y 16 regiones. No incluye Antártica (`12202`), código que no aparece en estos egresos. Una correspondencia no encontrada conserva el texto de origen y queda señalada para revisión.

| Columna | Qué significa y cómo se construye | Ejemplo ilustrativo e interpretación | Para qué sirve y qué precaución considerar |
| --- | --- | --- | --- |
| `GLOSA_COMUNA_RESIDENCIA_CORREGIDA` | Nombre obtenido al cruzar `COMUNA_RESIDENCIA` con `CUT_COM`. Si no hay correspondencia, conserva la glosa existente. | El código `10403` permite obtener `Hualaihué` desde `Hualaihu�`. | Mostrar nombres legibles en tablas, filtros y mapas. Consultar su estado: el campo también puede conservar categorías especiales o un texto no resuelto. |
| `GLOSA_REGION_RESIDENCIA_CORREGIDA` | Nombre obtenido al cruzar `REGION_RESIDENCIA` con `CUT_REG`, con el mismo criterio de conservación. | `De Los Lagos` pasa a `Los Lagos` según la denominación de referencia. | Presentar nombres consistentes. Una diferencia de denominación no significa necesariamente que el texto original estuviera mal codificado. |
| `ESTADO_CORRECCION_COMUNA` | Describe el resultado del cruce de comuna, considerando código, glosa, coincidencia y presencia de `�`. | `CORREGIDA` indica que se reparó una glosa dañada con un código conocido. | Diferenciar reparaciones, coincidencias y casos no resueltos. Es una categoría de control, no una escala de calidad numérica. |
| `ESTADO_CORRECCION_REGION` | Aplica las mismas reglas al código y nombre de región. | `NORMALIZADA` indica un cambio de nombre sin que hubiera `�`. | Interpretar los cambios regionales y revisar excepciones. No debe suponerse que todos los cambios fueron errores de codificación. |

### Estados de la corrección

Las reglas siguientes se aplican a ambos campos. Los ejemplos de esta tabla son ilustrativos; los conteos corresponden a resultados observados.

| Estado | Interpretación y ejemplo | Comuna: registros observados | Región: registros observados |
| --- | --- | ---: | ---: |
| `CORREGIDA` | Había `�` y se obtuvo un nombre distinto mediante el código; por ejemplo, `Hualaihu� → Hualaihué`. | 400.569 | 530.319 |
| `NORMALIZADA` | El nombre cambió para coincidir con la referencia, sin contener `�`; por ejemplo, `De Los Lagos → Los Lagos`. | 0 | 393.383 |
| `COINCIDE` | La glosa ya era igual al nombre de referencia. | 1.169.012 | 645.879 |
| `CATEGORIA_ESPECIAL` | El código representa `Ignorada` o `Extranjero`; se mantiene esa etiqueta. | 16.972 | 16.972 |
| `SIN_DATO` | Falta el código territorial; no se asigna un territorio. | 118.961 | 118.961 |
| `SIN_CORRESPONDENCIA` | Hay un código no especial que no aparece en el catálogo. Es una posibilidad prevista, no un resultado observado actualmente. | 0 | 0 |

Las categorías especiales corresponden a comuna `99999` y región `99` para `Ignorada`, y comuna `88888` y región literal `NULL` para `Extranjero`. Ese `NULL` no se convierte automáticamente en ausente.

**Ejemplo observado:** los 621 egresos con `COMUNA_RESIDENCIA=10403` conservan `Hualaihu�` en la glosa de origen y tienen `Hualaihué` en la corregida, con estado `CORREGIDA`. La lectura continúa siendo UTF-8: la letra se reconstruyó mediante el código, porque el CSV ya contenía `�`.

## 3. Información de diagnósticos CIE-10

`DIAG1` es el diagnóstico principal de egreso. Los códigos se preparan sin espacios externos, en mayúsculas y sin puntos. La categoría se obtiene del propio código; las descripciones y el capítulo se incorporan desde el diccionario entregado. `DIAG2` tiene un papel distinto: registra la causa externa.

| Columna | Qué significa y cómo se construye | Ejemplo ilustrativo e interpretación | Para qué sirve y qué precaución considerar |
| --- | --- | --- | --- |
| `DIAG1_CATEGORIA` | Agrupación formada por los tres primeros caracteres de `DIAG1` normalizado. Se obtiene aunque el código completo no aparezca en el diccionario. | `S720 → S72`. | Resumir diagnósticos con más detalle que un capítulo y menos que el código completo. El prefijo por sí solo no confirma correspondencia con el diccionario. |
| `DIAG1_DESCRIPCION` | Nombre de la subcategoría obtenido por coincidencia exacta del código completo. | Para `S720`, `FRACTURA DEL CUELLO DEL FÉMUR`. | Interpretar códigos y explicar resultados. Un ausente indica falta de descripción en ese cruce; no prueba que el diagnóstico sea inválido. |
| `DIAG1_GLOSA_CATEGORIA` | Nombre de la categoría obtenido al buscar `DIAG1_CATEGORIA` en el diccionario. | Para `S72`, `Fractura del fémur`. | Etiquetar agrupaciones diagnósticas. Es descriptiva del mismo concepto que el código de categoría; usar ambos como entradas puede duplicar información. |
| `DIAG1_CAPITULO` | Intervalo de códigos que identifica el capítulo asociado a la categoría en el diccionario. | `S00-T98` es un capítulo más amplio que `S72`. | Comparar grandes grupos diagnósticos. Es un código categórico, no una cantidad, y contiene información jerárquica relacionada con la categoría. |
| `DIAG1_GLOSA_CAPITULO` | Nombre del capítulo obtenido mediante la categoría diagnóstica. | Para `S00-T98`, `Traumatismos, envenenamientos y algunas otras consecuencias de causas externas`. | Presentar grupos amplios de forma comprensible. No es un nuevo diagnóstico y puede ser redundante con el código de capítulo. |
| `DIAG1_COINCIDENCIA` | Indica si el código completo está en el diccionario, si solo coincide la categoría o si no hay correspondencia o dato. La coincidencia exacta tiene prioridad. | `SOLO_CATEGORIA` permite interpretar la agrupación sin inventar la descripción del código completo. | Auditar el enriquecimiento y sus limitaciones. Una coincidencia confirma presencia en el diccionario, no exactitud clínica del registro. |
| `DIAG2_COINCIDENCIA` | Comprueba la presencia exacta del código de causa externa normalizado; distingue código no encontrado y dato ausente. No aplica búsqueda alternativa por categoría. | `SIN_DATO` indica que no hay un código disponible para buscar. | Revisar la cobertura del diccionario para causas externas. No es una medida de enfermedades concomitantes ni un indicador automático de error. |

### Estados de coincidencia diagnóstica

| Estado | Interpretación | DIAG1: registros observados | DIAG2: registros observados |
| --- | --- | ---: | ---: |
| `SUBCATEGORIA_EXACTA` | El código completo aparece en el diccionario. | 1.705.231 | 164.580 |
| `SOLO_CATEGORIA` | El código completo no aparece, pero sí su categoría de tres caracteres. | 251 | No se utiliza en este campo |
| `SIN_COINCIDENCIA` | Hay un código disponible, pero no se obtuvo la correspondencia prevista. | 32 | 0 |
| `SIN_DATO` | No hay código disponible para realizar el cruce. | 0 | 1.540.934 |

No debe confundirse `SIN_DATO`, una etiqueta de estado, con el ausente de una descripción. En 2025 hay **283 valores ausentes en `DIAG1_DESCRIPCION`**, de los cuales 251 corresponden a coincidencia solo de categoría y 32 a falta de correspondencia.

Una coincidencia exacta tampoco garantiza que todas las glosas complementarias estén completas. **Ejemplo observado:** `U069` tiene coincidencia exacta y capítulo `U00-U99`, pero la glosa de su categoría está ausente en el diccionario utilizado.

### Ejemplo observado de la jerarquía CIE-10

Esta combinación de valores aparece en la base; describe una correspondencia diagnóstica, no una historia clínica individual.

| Campo | Valor observado |
| --- | --- |
| `DIAG1` | `S720` |
| `DIAG1_CATEGORIA` | `S72` |
| `DIAG1_DESCRIPCION` | FRACTURA DEL CUELLO DEL FÉMUR |
| `DIAG1_GLOSA_CATEGORIA` | Fractura del fémur |
| `DIAG1_CAPITULO` | `S00-T98` |
| `DIAG1_GLOSA_CAPITULO` | Traumatismos, envenenamientos y algunas otras consecuencias de causas externas |
| `DIAG1_COINCIDENCIA` | `SUBCATEGORIA_EXACTA` |

## 4. Control de estadías extremas

| Columna | Qué significa y cómo se construye | Ejemplo ilustrativo e interpretación | Para qué sirve y qué precaución considerar |
| --- | --- | --- | --- |
| `ESTADIA_MAYOR_365` | Indica si `DIAS_ESTADA > 365`. La comparación ausente se completa con `False` en la implementación actual. | 365 días produce `False`; 366 días produce `True`. | Localizar casos para revisión y análisis de sensibilidad. El umbral es operativo, no una definición clínica universal. La marca no elimina ni recorta valores. `False` no sustituye revisar si los días son válidos o están ausentes. |

En 2025 hay 637 valores `True`. Las estadías marcadas van de 366 a 20.368 días.

**No utilizar `ESTADIA_MAYOR_365` como predictor al estimar `DIAS_ESTADA`: se calcula directamente a partir de lo que se quiere estimar y produciría fuga de información.**

## 5. Uso en las tres líneas de investigación

Cada línea elegirá en su notebook los atributos de la base compartida que sirvan para su análisis.

| Línea | Cómo pueden aprovecharse las auxiliares | Qué evitar o revisar |
| --- | --- | --- |
| Clasificación de `CONDICION_EGRESO` | Usar categorías o capítulos diagnósticos como representaciones posibles, nombres geográficos para interpretar resultados y marcas de calidad para estudiar diferencias de cobertura. | Excluir el identificador y las variables de control como entradas automáticas. Verificar qué información estaría disponible en el momento de uso; duración y condición de egreso se conocen al finalizar la hospitalización. |
| Regresión de `DIAS_ESTADA` | Comparar errores por categoría diagnóstica, territorio o calidad de información; usar la marca de estadía extrema para describir después dónde falla el modelo. | Excluir `ESTADIA_MAYOR_365` de los predictores. Evitar también indicadores de control construidos con los resultados, como `EN_GRUPO_REPETIDO`, sin una justificación específica. |
| Clustering y reglas de asociación | Elegir una representación diagnóstica y territorial para describir perfiles; utilizar las glosas como etiquetas legibles y las marcas para revisar la calidad de los grupos o reglas. | Para construir perfiles y comparar después duración y mortalidad, dejar ambos resultados y sus derivados fuera del agrupamiento inicial. Evitar reglas triviales entre un código y su glosa o entre una variable y su derivación directa. |

Como criterio práctico, elegir entre código y glosa del mismo concepto y justificar si se combinan distintos niveles CIE-10. Las marcas de calidad describen el registro y su procesamiento; no son automáticamente características clínicas. El diagnóstico principal corresponde al egreso, por lo que su uso retrospectivo no demuestra capacidad de predicción al ingreso.

Las imputaciones estadísticas, codificaciones, particiones y demás preparaciones de modelos se definirán después, respetando la separación entre entrenamiento y evaluación. Este documento describe las columnas existentes; no incorpora nuevas transformaciones ni resultados de modelos.
