# Preprocesamiento común de egresos hospitalarios 2023–2025

`01_preprocesamiento_egresos_2025.ipynb` aplica el mismo preprocesamiento a los egresos de 2023, 2024 y 2025. La base común sirve de punto de partida para clasificación, regresión lineal, clustering y reglas de asociación.

## Ejecutar

El entorno `.venv` no se incluye en GitHub. Cada integrante debe crearlo con Python 3.13 e instalar las dependencias de `requirements.txt`. Para trabajar en VS Code también necesita las extensiones Python y Jupyter.

Para trabajar en VS Code:

1. Colocar `EGRESOS_2023.zip`, `EGRESOS_2024.zip` y `EGRESOS_2025.zip` en `datos/originales/` y abrir el notebook.
2. Pulsar **Seleccionar kernel** y elegir **.venv** en **Entornos de Python**.
3. Pulsar **Ejecutar todo**. El notebook procesa los ZIP de 2023, 2024 y 2025 en orden y muestra la exploración de cada año.

No es necesario activar el entorno desde PowerShell para ejecutar el notebook con ese kernel. Para recrear el entorno en otro equipo con Python 3.13 instalado:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

El notebook escribe cada año en un único Parquet temporal. Una vez verificados el esquema y los 4.985.130 egresos distribuidos por año, reemplaza la base común. Si la ejecución se interrumpe o falla, la última base válida sigue disponible. Los informes anuales, los ZIP y el Parquet se guardan localmente y están excluidos de Git.

## Organización

- Los ZIP de `datos/originales/` contienen el CSV y el diccionario de cada año.
- `datos/procesados/egresos_2023_2025_base_comun.parquet` reúne los tres años para el equipo.
- `resultados/egresos_<año>/tablas/` guarda el diccionario, la calidad de datos, las transformaciones, los diagnósticos pendientes y el resumen de procesamiento de cada año.
- `resultados/egresos_<año>/figuras/` guarda los tres gráficos descriptivos. `resultados/tablas/` y `resultados/figuras/` contienen los resultados de la versión anterior de 2025.

Lectura de la base compartida:

```python
import pandas as pd
base = pd.read_parquet("datos/procesados/egresos_2023_2025_base_comun.parquet")
```

## Criterios comunes

La base común reúne los 16 campos compartidos, `ERROR` y todos los egresos. Las repeticiones se cuentan dentro de cada archivo con los 16 campos compartidos. Se conservan los registros repetidos y las estadías extremas.

`ERROR` conserva los valores del CSV de 2025. En 2023 y 2024 queda como `null` porque esos archivos no traen el campo. El resumen de 2025 registra el recuento de 0 y 1. La columna queda disponible para revisar la calidad de los datos mientras se aclara su significado.

`ANO_FUENTE` registra el año indicado por el ZIP. Junto con `ID_REGISTRO`, el número de fila dentro de su CSV, permite ubicar cada registro. En `GRUPO_EDAD` se unifican las variantes de «Menor de 1 año» y se registran los cambios por año en `transformaciones.csv`. Las glosas geográficas corregidas se guardan en columnas auxiliares junto con las de origen.

Los vacíos y asteriscos se representan como ausentes, con marcas de origen. No se atribuye una causa a los marcadores ni se imputan valores. Se conserva el texto `NULL` en región porque los tres archivos lo asocian con la glosa `Extranjero`. Los vacíos de `DIAG2`, causa externa, pueden ser estructurales. Los códigos sin correspondencia en el diccionario se conservan para revisión.

El conteo de filas modificadas considera cambios de contenido o representación de faltantes en los campos originales. No incluye la mera adición de atributos ni una conversión de tipo que conserva el mismo valor textual. Los conteos de distintas transformaciones se pueden superponer.

La preparación por algoritmo, selección de entradas y objetivos, partición, codificación, escalamiento y entrenamiento se realizarán después, partiendo de esta misma base. No se crean los notebooks específicos en esta etapa.

## Documentación

- [Diccionario de columnas auxiliares de la base procesada](docs/DICCIONARIO_BASE_PROCESADA.md)

## Archivos incluidos y datos de entrada

El repositorio incluye el notebook, las dependencias, la documentación, el catálogo geográfico y los resultados de la versión anterior. Cada ZIP debe contener `Diccionario BD egresos hospitalario.xlsx` y su CSV. Los nombres son `EGRESOS_2023.csv`, `EGR_DATOS_ABIERTO_2024.csv` y `EGR_DATOS_ABIERTO_2025.csv`.

## Reglas compartidas del equipo

`AGENTS.md` reúne las reglas de programación del proyecto y `.agents/skills/kill-slop/` contiene la skill compartida para mantener el código sencillo y legible. Se incluyen en Git para que los integrantes trabajen con las mismas instrucciones.

Estas instrucciones no afectan la ejecución de Python. Codex puede descubrirlas en la carpeta del repositorio; otras herramientas pueden requerir configuración propia. Al solicitar cambios a un asistente, indicar que respete `AGENTS.md` y utilice `$kill-slop`.
