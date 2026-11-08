# Preprocesamiento común de egresos hospitalarios 2025

`01_preprocesamiento_egresos_2025.ipynb` explora y prepara una sola base para las tres líneas del equipo: clasificación, regresión lineal, clustering y reglas de asociación. Se reutiliza la exploración anterior y se retira la preparación específica de regresión.

## Ejecutar

El entorno `.venv` no se incluye en GitHub. Cada integrante debe crearlo con Python 3.13 e instalar las dependencias de `requirements.txt`. Para trabajar en VS Code también necesita las extensiones Python y Jupyter.

Para trabajar en VS Code:

1. Abrir esta carpeta y el notebook.
2. Pulsar **Seleccionar kernel**, arriba a la derecha. Si hace falta, elegir **Seleccionar otro kernel > Entornos de Python** y seleccionar **.venv (Python 3.13.6)**.
3. Pulsar **Ejecutar todo**, o ejecutar las celdas en orden.

No es necesario activar el entorno desde PowerShell para ejecutar el notebook con ese kernel. Para recrear el entorno en otro equipo con Python 3.13 instalado:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

La versión del notebook publicada en GitHub no incluye salidas de ejecución guardadas, para evitar publicar vistas de registros hospitalarios. **Ejecutar todo** genera las salidas localmente usando el entorno seleccionado. Las tablas y figuras agregadas están disponibles en `resultados/`.

## Organización

- `datos/originales/EGRESOS_2025.zip`: fuente intacta, con CSV y diccionario.
- `datos/procesados/egresos_2025_base_comun.parquet`: única base procesada, compartida por todo el equipo.
- `resultados/tablas/descripcion_variables.csv`: descripción y tipos de todos los atributos finales.
- `resultados/tablas/calidad_datos.csv`: marcadores originales, ausentes finales y cambios por variable.
- `resultados/tablas/transformaciones.csv`: motivo y cantidad de celdas y filas afectadas por cada transformación.
- `resultados/tablas/diagnosticos_sin_coincidencia_exacta.csv`: códigos pendientes de revisión.
- `resultados/tablas/resumen_procesamiento.csv`: conteos y verificación de conservación de datos.
- `resultados/figuras/`: tres gráficos descriptivos. Las restantes tablas exploratorias se consultan en el notebook.

Lectura de la base compartida:

```python
import pandas as pd
base = pd.read_parquet("datos/procesados/egresos_2025_base_comun.parquet")
```

## Criterios comunes

Se mantienen las 17 variables originales, incluidas `CONDICION_EGRESO` y `DIAS_ESTADA`, y todos los egresos. Se conservan las repeticiones, las estadías extremas y los registros con `ERROR=1`.

Los vacíos y asteriscos se representan como ausentes, con marcas de origen. No se atribuye una causa a los marcadores ni se imputan valores. El literal `NULL` en región se conserva: en este archivo se asocia a la glosa `Extranjero`. Los vacíos de `DIAG2`, causa externa, pueden ser estructurales. Los códigos sin correspondencia en el diccionario no se declaran inválidos ni se eliminan.

El conteo de filas modificadas considera cambios de contenido o representación de faltantes en los campos originales. No incluye la mera adición de atributos ni una conversión de tipo que conserva el mismo valor textual. Los conteos de distintas transformaciones se pueden superponer.

La preparación por algoritmo, selección de entradas y objetivos, partición, codificación, escalamiento y entrenamiento se realizarán después, partiendo de esta misma base. No se crean los notebooks específicos en esta etapa.

## Documentación

- [Diccionario de columnas auxiliares de la base procesada](docs/DICCIONARIO_BASE_PROCESADA.md)

## Archivos incluidos y datos de entrada

El repositorio incluye el notebook, las dependencias declaradas, la documentación, las tablas y figuras generadas y el catálogo geográfico pequeño. No incluye `.venv`, archivos temporales, el ZIP original ni la base Parquet generada.

Para recalcular el proyecto después de clonarlo, colocar `EGRESOS_2025.zip` en `datos/originales/`. Debe contener `EGR_DATOS_ABIERTO_2025.csv` y `Diccionario BD egresos hospitalario.xlsx`, los archivos originales utilizados por el equipo. Después, crear el entorno, instalar las dependencias y ejecutar el notebook. El proceso generará la base común en `datos/procesados/`.

Los enlaces a los datos dentro de la documentación corresponden a archivos locales: estarán disponibles una vez colocada la fuente y ejecutado el preprocesamiento.

## Reglas compartidas del equipo

`AGENTS.md` reúne las reglas de programación del proyecto y `.agents/skills/kill-slop/` contiene la skill compartida para mantener el código sencillo y legible. Se incluyen en Git para que los integrantes trabajen con las mismas instrucciones.

Estas instrucciones no afectan la ejecución de Python. Codex puede descubrirlas en la carpeta del repositorio; otras herramientas pueden requerir configuración propia. Al solicitar cambios a un asistente, indicar que respete `AGENTS.md` y utilice `$kill-slop`.
