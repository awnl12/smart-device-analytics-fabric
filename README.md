# Smart Device Analytics Platform con Microsoft Fabric

Proyecto end-to-end de **ingeniería de datos y analítica** desarrollado con **Microsoft Fabric**, implementando una arquitectura tipo **Medallion Architecture** con capas **Bronze, Silver y Gold**.

El proyecto utiliza un dataset de dispositivos inteligentes que contiene información histórica sobre smartphones, tablets, wearables y otros dispositivos. La solución permite cargar, transformar, modelar, orquestar y visualizar datos utilizando herramientas modernas del ecosistema de Microsoft Fabric.

Este proyecto fue desarrollado con fines académicos y de práctica profesional, complementando la implementación con resolución de errores reales, parametrización de notebooks, corrección de esquemas, manejo básico de errores, orquestación de pipelines y prácticas de deployment entre workspaces.

---

## Tabla de contenidos

- [Objetivo del proyecto](#objetivo-del-proyecto)
- [Dataset utilizado](#dataset-utilizado)
- [Arquitectura de la solución](#arquitectura-de-la-solución)
- [Tecnologías utilizadas](#tecnologías-utilizadas)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Flujo general del proyecto](#flujo-general-del-proyecto)
- [Capa Bronze](#capa-bronze)
- [Capa Silver](#capa-silver)
- [Capa Gold](#capa-gold)
- [Ingesta de datos](#ingesta-de-datos)
- [Transformación de datos](#transformación-de-datos)
- [Orquestación con Pipelines](#orquestación-con-pipelines)
- [Manejo de errores](#manejo-de-errores)
- [Modelo semántico](#modelo-semántico)
- [Power BI Report y Dashboard](#power-bi-report-y-dashboard)
- [Task Flow](#task-flow)
- [Lineage View](#lineage-view)
- [Deployment Pipelines](#deployment-pipelines)
- [Complicaciones encontradas y soluciones](#complicaciones-encontradas-y-soluciones)
- [Resultados obtenidos](#resultados-obtenidos)
- [Aprendizajes principales](#aprendizajes-principales)
- [Mejoras futuras](#mejoras-futuras)
- [Autor](#autor)

---

## Objetivo del proyecto

El objetivo principal fue construir una solución completa de datos usando **Microsoft Fabric**, desde la carga inicial de archivos hasta la creación de reportes analíticos en Power BI.

La solución cubre el siguiente flujo:

```text
Kaggle Dataset
      |
      v
Azure Data Lake Storage Gen2
      |
      v
OneLake Shortcut
      |
      v
Bronze Lakehouse
      |
      v
Ingestion Notebooks con PySpark
      |
      v
Silver Lakehouse
      |
      v
Transformation Notebooks / Dataflows Gen2
      |
      v
Gold Lakehouse / Warehouse
      |
      v
Semantic Model
      |
      v
Power BI Report / Dashboard
```

El proyecto incluye:

- Carga del dataset hacia Azure Data Lake Storage Gen2.
- Uso de Azure Storage Explorer para cargar archivos al Data Lake.
- Conexión entre ADLS Gen2 y Microsoft Fabric mediante OneLake Shortcut.
- Creación de Lakehouses para organizar datos por capas.
- Procesamiento de datos con notebooks y PySpark.
- Transformaciones con Dataflows Gen2 y Power Query.
- Implementación de cargas completas e incrementales.
- Orquestación con Data Factory Pipelines.
- Parametrización de procesos con `file_date` y `environment`.
- Validación de carpetas antes de ejecutar la ingesta.
- Notificación por correo electrónico en caso de error.
- Creación de modelo semántico.
- Creación de reporte y dashboard en Power BI.
- Organización del proyecto con Task Flow.
- Análisis de dependencias con Lineage View.
- Práctica de despliegue mediante Deployment Pipelines.

---

## Dataset utilizado

El dataset utilizado corresponde a información de dispositivos inteligentes.

Contiene información relacionada con más de 24,000 dispositivos, incluyendo:

- Smartphones
- Tablets
- Wearables
- Modelos
- Marcas
- Categorías
- Cámaras
- Conectividad
- Sistemas operativos
- Pantallas
- Especificaciones físicas

Fuente del dataset:

```text
https://www.kaggle.com/datasets/elilmaaac/smart-devices-from-99s-to-today
```

El dataset fue descargado desde Kaggle y cargado manualmente hacia **Azure Data Lake Storage Gen2** utilizando **Azure Storage Explorer**.

Los archivos fueron organizados dentro del contenedor del proyecto en carpetas por fecha de procesamiento.

Ejemplo:

```text
smart-device/
└── bronze-data/
    ├── 2025-07-07/
    ├── 2025-07-14/
    └── 2025-07-21/
```

Posteriormente, esta ubicación fue conectada con Microsoft Fabric mediante un **OneLake Shortcut**, permitiendo que el Lakehouse Bronze acceda a los archivos almacenados en ADLS Gen2.

### Evidencia del dataset y carga inicial

📌 **IMAGEN 01 - FUENTE DEL DATASET EN KAGGLE**  
Guardar la imagen como:

```text
docs/images/01_dataset_kaggle.png
```

Pegar aquí:

![Fuente del dataset en Kaggle](docs/images/01_dataset_kaggle.png)

📌 **IMAGEN 02 - AZURE STORAGE EXPLORER CON LOS ARCHIVOS CARGADOS**  
Guardar la imagen como:

```text
docs/images/02_azure_storage_explorer.png
```

Pegar aquí:

![Carga de archivos con Azure Storage Explorer](docs/images/02_azure_storage_explorer.png)

📌 **IMAGEN 03 - CONTENEDOR EN AZURE DATA LAKE STORAGE GEN2 CON CARPETAS POR FECHA**  
Esta imagen debe mostrar carpetas como `2025-07-07`, `2025-07-14`, `2025-07-21`.  
Guardar la imagen como:

```text
docs/images/03_adls_bronze_folders.png
```

Pegar aquí:

![Carpetas Bronze en ADLS Gen2](docs/images/03_adls_bronze_folders.png)

---

## Arquitectura de la solución

La solución fue diseñada siguiendo una arquitectura Medallion.

La arquitectura Medallion permite organizar los datos en diferentes capas según su nivel de procesamiento:

- **Bronze:** datos crudos.
- **Silver:** datos limpios, estructurados y normalizados.
- **Gold:** datos preparados para análisis, reportes y modelos semánticos.

Flujo conceptual:

```text
Data Source
    |
    v
Azure Data Lake Storage Gen2
    |
    v
OneLake Shortcut
    |
    v
Bronze Lakehouse
    |
    v
Ingestion Notebooks
    |
    v
Silver Lakehouse
    |
    v
Transformation Notebooks / Dataflows Gen2
    |
    v
Gold Lakehouse / Warehouse
    |
    v
Semantic Model
    |
    v
Power BI
```

📌 **IMAGEN 04 - DIAGRAMA DE ARQUITECTURA DEL PROYECTO**  
Esta imagen debe mostrar el flujo completo desde el origen de datos hasta Power BI.  
Puedes hacer un diagrama propio en draw.io, PowerPoint, Canva o usar una captura propia si la tienes.  
Guardar la imagen como:

```text
docs/images/04_architecture_diagram.png
```

Pegar aquí:

![Arquitectura del proyecto](docs/images/04_architecture_diagram.png)

---

## Tecnologías utilizadas

En este proyecto se utilizaron las siguientes tecnologías y componentes:

| Tecnología | Uso dentro del proyecto |
|---|---|
| Microsoft Fabric | Plataforma principal del proyecto |
| Azure Data Lake Storage Gen2 | Almacenamiento externo del dataset |
| Azure Storage Explorer | Carga manual de archivos hacia ADLS Gen2 |
| OneLake | Capa de almacenamiento unificada de Fabric |
| OneLake Shortcut | Conexión entre Fabric y ADLS Gen2 |
| Lakehouse | Almacenamiento de datos en capas Bronze, Silver y Gold |
| Data Factory Pipelines | Orquestación de ingesta y transformación |
| Fabric Notebooks | Desarrollo de procesos de ingesta y transformación |
| PySpark | Procesamiento de datos distribuido |
| Spark DataFrames | Lectura, limpieza y transformación de datos |
| Delta Lake | Escritura de tablas Delta |
| Dataflows Gen2 | Transformaciones visuales con Power Query |
| Power Query | Transformación de datos en Dataflows Gen2 |
| Semantic Model | Modelo analítico para Power BI |
| Power BI | Reportes y dashboard |
| Task Flow | Organización visual del proyecto |
| Lineage View | Visualización de dependencias |
| Deployment Pipelines | Despliegue entre workspaces |

---

## Estructura del repositorio

Estructura sugerida del repositorio:

```text
smart-device-analytics-fabric/
│
├── notebooks/
│   ├── ingestion/
│   │   ├── 01_Ingestion_file_service.ipynb
│   │   ├── 02_Ingestion_file_model.ipynb
│   │   ├── 03_Ingestion_file_category.ipynb
│   │   ├── 04_Ingestion_file_camera.ipynb
│   │   ├── 05_Ingestion_file_brand.ipynb
│   │   ├── 06_Ingestion_file_operating_system.ipynb
│   │   ├── 07_Ingestion_file_connectivity.ipynb
│   │   ├── 08_Ingestion_file_display.ipynb
│   │   └── 09_Ingestion_file_physical_specs.ipynb
│   │
│   └── transformation/
│       ├── 01_Transformation_table_category.ipynb
│       ├── 02_Transformation_table_device.ipynb
│       ├── 03_Transformation_table_model.ipynb
│       ├── 04_Transformation_table_display.ipynb
│       ├── 05_Transformation_table_physical_specs.ipynb
│       └── 06_Transformation_dim_time.ipynb
│
├── dataflows/
│   ├── DF_Transformation_Brand.md
│   ├── DF_Transformation_Camera.md
│   ├── DF_Transformation_Connectivity.md
│   └── DF_Transformation_Operating_System.md
│
├── pipelines/
│   ├── pl_ingest_smart_device.md
│   ├── pl_transformation_smart_device.md
│   └── pl_main_smart_device.md
│
├── powerbi/
│   ├── semantic_model_notes.md
│   └── report_screenshots.md
│
├── docs/
│   └── images/
│       ├── 01_dataset_kaggle.png
│       ├── 02_azure_storage_explorer.png
│       ├── 03_adls_bronze_folders.png
│       ├── 04_architecture_diagram.png
│       ├── 05_workspace_fabric.png
│       ├── 06_task_flow.png
│       ├── 07_ingestion_pipeline.png
│       ├── 08_ingestion_pipeline_success.png
│       ├── 09_transformation_pipeline.png
│       ├── 10_transformation_pipeline_success.png
│       ├── 11_main_pipeline.png
│       ├── 12_main_pipeline_success.png
│       ├── 13_lakehouse_silver_tables.png
│       ├── 14_lakehouse_gold_tables.png
│       ├── 15_semantic_model.png
│       ├── 16_powerbi_report.png
│       ├── 17_dashboard.png
│       ├── 18_lineage_view.png
│       └── 19_deployment_pipeline.png
│
├── README.md
└── LICENSE
```

---

## Flujo general del proyecto

El flujo general del proyecto fue dividido en tres grandes etapas:

1. **Data Source**
2. **Prepare and Transform**
3. **Analyze**

Flujo implementado:

```text
Kaggle Dataset
      |
      v
Azure Data Lake Storage Gen2
      |
      v
OneLake Shortcut
      |
      v
Bronze Lakehouse
      |
      v
Ingestion Process
      |
      v
Silver Lakehouse
      |
      v
Transformation Process
      |
      v
Gold Lakehouse / Warehouse
      |
      v
Semantic Model
      |
      v
Power BI Report / Dashboard
```

📌 **IMAGEN 05 - WORKSPACE DE FABRIC CON LOS ELEMENTOS DEL PROYECTO**  
Esta imagen debe mostrar el área de trabajo de Microsoft Fabric con elementos como Lakehouses, Pipelines, Dataflows, Notebooks, Semantic Model, Report o Dashboard.  
Guardar la imagen como:

```text
docs/images/05_workspace_fabric.png
```

Pegar aquí:

![Workspace de Microsoft Fabric](docs/images/05_workspace_fabric.png)

---

## Capa Bronze

La capa **Bronze** almacena los datos en su formato original.

En este proyecto, la capa Bronze se encuentra en **Azure Data Lake Storage Gen2**. Los archivos fueron cargados mediante **Azure Storage Explorer** y organizados por fechas de procesamiento.

Ejemplo de estructura:

```text
smart-device/
└── bronze-data/
    ├── 2025-07-07/
    ├── 2025-07-14/
    └── 2025-07-21/
```

Cada carpeta contiene los archivos necesarios para procesar una ejecución específica del pipeline.

La capa Bronze fue conectada a Microsoft Fabric mediante un **OneLake Shortcut**, permitiendo acceder a los archivos desde el Lakehouse sin duplicar físicamente los datos.

### Actividades realizadas en Bronze

- Descarga del dataset desde Kaggle.
- Creación de cuenta de almacenamiento en Azure.
- Creación de contenedor para el proyecto.
- Carga del dataset mediante Azure Storage Explorer.
- Organización de archivos por fecha.
- Creación de Shortcut en OneLake hacia ADLS Gen2.
- Preparación del Lakehouse Bronze en Fabric.

📌 **IMAGEN 06 - RECURSO DE AZURE / RESOURCE GROUP DEL PROYECTO**  
Esta imagen debe mostrar el grupo de recursos en Azure y la cuenta de almacenamiento usada para el proyecto.  
Antes de subirla, tapa el correo, subscription ID y cualquier dato sensible.  
Guardar la imagen como:

```text
docs/images/06_azure_resource_group.png
```

Pegar aquí:

![Resource Group en Azure](docs/images/06_azure_resource_group.png)

📌 **IMAGEN 07 - CARPETAS DE BRONZE EN AZURE DATA LAKE STORAGE GEN2**  
Esta imagen debe mostrar el contenedor y las carpetas por fecha.  
Guardar la imagen como:

```text
docs/images/07_bronze_folders_adls.png
```

Pegar aquí:

![Carpetas Bronze en ADLS Gen2](docs/images/07_bronze_folders_adls.png)

---

## Capa Silver

La capa **Silver** contiene datos más limpios, estructurados y preparados para siguientes transformaciones.

En esta etapa se utilizaron **notebooks con PySpark** para leer los archivos de Bronze, aplicar esquemas, seleccionar columnas, renombrar campos y escribir tablas Delta en el Lakehouse Silver.

### Procesos aplicados en Silver

- Lectura de archivos desde la capa Bronze.
- Definición de schemas.
- Selección de columnas necesarias.
- Renombrado de columnas.
- Conversión de tipos de datos.
- Inclusión de columnas de auditoría.
- Inclusión de `ingestion_date`.
- Inclusión de `file_date`.
- Inclusión de `environment`.
- Escritura de datos en formato Delta.

### Archivos procesados en la ingesta

Los notebooks de ingesta procesaron los siguientes archivos:

| Archivo | Descripción |
|---|---|
| Device | Información principal del dispositivo |
| Model | Información de modelos |
| Category | Categorías de dispositivos |
| Camera | Características de cámara |
| Brand | Información de marcas |
| Operating System | Sistemas operativos |
| Connectivity | Información de conectividad |
| Display | Características de pantalla |
| Physical Specs | Especificaciones físicas |

📌 **IMAGEN 08 - LAKEHOUSE SILVER CON TABLAS GENERADAS**  
Esta imagen debe mostrar las tablas generadas en la capa Silver.  
Guardar la imagen como:

```text
docs/images/08_lakehouse_silver_tables.png
```

Pegar aquí:

![Tablas en Lakehouse Silver](docs/images/08_lakehouse_silver_tables.png)

---

## Capa Gold

La capa **Gold** contiene los datos preparados para análisis y consumo desde Power BI.

En esta capa se generaron tablas finales con estructura analítica, incluyendo dimensiones, tabla de hechos y dimensión de tiempo.

### Tablas finales de análisis

Algunas de las tablas generadas fueron:

| Tabla | Tipo | Descripción |
|---|---|---|
| fact_device | Hechos | Tabla principal de análisis de dispositivos |
| dim_category | Dimensión | Categorías de dispositivos |
| dim_model | Dimensión | Modelos de dispositivos |
| dim_brand | Dimensión | Marcas |
| dim_camera | Dimensión | Características de cámara |
| dim_display | Dimensión | Características de pantalla |
| dim_operating_system | Dimensión | Sistemas operativos |
| dim_physical_specs | Dimensión | Especificaciones físicas |
| dim_time | Dimensión | Dimensión de calendario |

📌 **IMAGEN 09 - LAKEHOUSE GOLD O WAREHOUSE CON TABLAS FINALES**  
Esta imagen debe mostrar las tablas finales de la capa Gold o Warehouse.  
Guardar la imagen como:

```text
docs/images/09_lakehouse_gold_tables.png
```

Pegar aquí:

![Tablas finales en Gold](docs/images/09_lakehouse_gold_tables.png)

---

## Ingesta de datos

La ingesta fue desarrollada con **Microsoft Fabric Notebooks** utilizando **PySpark**.

Cada notebook fue responsable de procesar un archivo específico desde la capa Bronze hacia la capa Silver.

### Flujo general de un notebook de ingesta

```text
Leer archivo desde Bronze
        |
        v
Aplicar schema definido
        |
        v
Seleccionar columnas necesarias
        |
        v
Renombrar columnas
        |
        v
Agregar columnas de control
        |
        v
Agregar ingestion_date
        |
        v
Agregar file_date y environment
        |
        v
Escribir tabla Delta en Silver
```

### Parametrización de notebooks

Los notebooks fueron parametrizados para poder ser ejecutados desde pipelines.

Parámetros principales:

```text
file_date
environment
```

Ejemplo:

```text
file_date = 2025-07-07
environment = Development
```

Esto permite reutilizar el mismo proceso para distintas fechas de carga y distintos ambientes.

### Notebooks de ingesta

| Notebook | Objetivo |
|---|---|
| Ingestion Device File | Ingestar archivo de dispositivos |
| Ingestion Model File | Ingestar archivo de modelos |
| Ingestion Category File | Ingestar archivo de categorías |
| Ingestion Camera File | Ingestar archivo de cámaras |
| Ingestion Brand File | Ingestar archivo de marcas |
| Ingestion Operating System File | Ingestar archivo de sistemas operativos |
| Ingestion Connectivity File | Ingestar archivo de conectividad |
| Ingestion Display File | Ingestar archivo de pantallas |
| Ingestion Physical Specs File | Ingestar archivo de especificaciones físicas |

📌 **IMAGEN 10 - NOTEBOOK DE INGESTA ABIERTO EN FABRIC**  
Esta imagen debe mostrar un notebook de ingesta con código PySpark.  
Guardar la imagen como:

```text
docs/images/10_ingestion_notebook.png
```

Pegar aquí:

![Notebook de ingesta](docs/images/10_ingestion_notebook.png)

---

## Transformación de datos

La transformación fue realizada utilizando una combinación de:

- Notebooks con PySpark.
- Dataflows Gen2.
- Power Query.

Los notebooks se utilizaron principalmente para transformaciones con lógica más controlada, mientras que los Dataflows Gen2 fueron usados para transformaciones visuales y directas.

### Transformaciones implementadas

| Proceso | Herramienta |
|---|---|
| Transformation Category | Notebook |
| Transformation Device | Notebook |
| Transformation Model | Notebook |
| Transformation Display | Notebook |
| Transformation Physical Specs | Notebook |
| Transformation Dim Time | Notebook |
| DF Transformation Brand | Dataflow Gen2 |
| DF Transformation Camera | Dataflow Gen2 |
| DF Transformation Connectivity | Dataflow Gen2 |
| DF Transformation Operating System | Dataflow Gen2 |

### Flujo general de transformación

```text
Leer datos desde Silver
        |
        v
Aplicar reglas de transformación
        |
        v
Limpiar columnas innecesarias
        |
        v
Aplicar lógica de carga completa o incremental
        |
        v
Generar dimensiones y tabla de hechos
        |
        v
Escribir tablas finales en Gold
```

📌 **IMAGEN 11 - NOTEBOOK DE TRANSFORMACIÓN ABIERTO EN FABRIC**  
Esta imagen debe mostrar un notebook de transformación.  
Guardar la imagen como:

```text
docs/images/11_transformation_notebook.png
```

Pegar aquí:

![Notebook de transformación](docs/images/11_transformation_notebook.png)

📌 **IMAGEN 12 - DATAFLOW GEN2 DE TRANSFORMACIÓN**  
Esta imagen debe mostrar un Dataflow Gen2 usado para transformar Brand, Camera, Connectivity u Operating System.  
Guardar la imagen como:

```text
docs/images/12_dataflow_gen2.png
```

Pegar aquí:

![Dataflow Gen2](docs/images/12_dataflow_gen2.png)

---

## Carga completa e incremental

Durante el proyecto se aplicaron patrones de diseño de carga de datos.

### Carga completa

La carga completa fue utilizada para tablas donde se puede reemplazar el contenido completo en cada ejecución.

Ejemplos:

- Algunas dimensiones pequeñas.
- Tablas de referencia.
- Catálogos.

### Carga incremental

La carga incremental fue utilizada para procesar datos asociados a una fecha específica y evitar reprocesar todo el histórico.

El parámetro principal para controlar este comportamiento fue:

```text
file_date
```

Ejemplo:

```text
file_date = 2025-07-07
```

Esto permite procesar únicamente la carpeta correspondiente a esa fecha en la capa Bronze.

---

## Orquestación con Pipelines

La orquestación fue implementada con **Data Factory Pipelines** dentro de Microsoft Fabric.

Se crearon tres pipelines principales:

| Pipeline | Descripción |
|---|---|
| pl_ingest_smart_device | Ejecuta el proceso de ingesta |
| pl_transformation_smart_device | Ejecuta el proceso de transformación |
| pl_main_smart_device | Orquesta el flujo completo invocando ingesta y transformación |

---

## Pipeline de ingesta

El pipeline de ingesta valida primero si existe la carpeta correspondiente a la fecha de procesamiento.

### Flujo del pipeline de ingesta

```text
Get Folder Details
        |
        v
If Condition
        |
        |-- True  → Ejecutar notebooks de ingesta
        |
        |-- False → Enviar email de error
```

Si la carpeta existe, se ejecutan los notebooks de ingesta.

Si la carpeta no existe, se ejecuta una actividad de notificación por correo.

📌 **IMAGEN 13 - PIPELINE DE INGESTA COMPLETO**  
Esta imagen debe mostrar el pipeline de ingesta con Get Folder Details, If Condition, notebooks y email de error.  
Guardar la imagen como:

```text
docs/images/13_ingestion_pipeline.png
```

Pegar aquí:

![Pipeline de ingesta](docs/images/13_ingestion_pipeline.png)

📌 **IMAGEN 14 - VALIDACIÓN IF CONDITION DEL PIPELINE DE INGESTA**  
Esta imagen debe mostrar claramente la condición `True` con los notebooks y la condición `False` con email de error.  
Guardar la imagen como:

```text
docs/images/14_ingestion_if_condition.png
```

Pegar aquí:

![Validación If Condition](docs/images/14_ingestion_if_condition.png)

📌 **IMAGEN 15 - EJECUCIÓN EXITOSA DEL PIPELINE DE INGESTA**  
Esta imagen debe mostrar las actividades de ingesta en estado correcto.  
Guardar la imagen como:

```text
docs/images/15_ingestion_pipeline_success.png
```

Pegar aquí:

![Ejecución exitosa de ingesta](docs/images/15_ingestion_pipeline_success.png)

---

## Pipeline de transformación

El pipeline de transformación ejecuta los procesos que generan la capa Gold.

Este pipeline incluye notebooks y Dataflows Gen2.

### Procesos dentro del pipeline de transformación

- Transformación de modelos.
- Transformación de especificaciones físicas.
- Transformación de pantallas.
- Transformación de sistemas operativos.
- Transformación de conectividad.
- Transformación de marcas.
- Transformación de cámaras.
- Transformación de categorías.
- Transformación de dispositivos.
- Creación de dimensión de tiempo.

📌 **IMAGEN 16 - PIPELINE DE TRANSFORMACIÓN COMPLETO**  
Esta imagen debe mostrar todos los notebooks y Dataflows Gen2 del pipeline de transformación.  
Guardar la imagen como:

```text
docs/images/16_transformation_pipeline.png
```

Pegar aquí:

![Pipeline de transformación](docs/images/16_transformation_pipeline.png)

📌 **IMAGEN 17 - EJECUCIÓN EXITOSA DEL PIPELINE DE TRANSFORMACIÓN**  
Esta imagen debe mostrar las actividades de transformación en verde o estado correcto.  
Guardar la imagen como:

```text
docs/images/17_transformation_pipeline_success.png
```

Pegar aquí:

![Ejecución exitosa de transformación](docs/images/17_transformation_pipeline_success.png)

---

## Pipeline principal

El pipeline principal automatiza el flujo completo del proyecto.

Este pipeline invoca primero el pipeline de ingesta y luego el pipeline de transformación.

### Flujo del pipeline principal

```text
Invoke Ingestion Pipeline
        |
        v
Invoke Transformation Pipeline
```

Ambos pipelines reciben los parámetros:

```text
file_date
environment
```

Esto permite mantener una misma fecha y ambiente durante toda la ejecución.

📌 **IMAGEN 18 - PIPELINE PRINCIPAL CON INVOCACIÓN DE INGESTA Y TRANSFORMACIÓN**  
Esta imagen debe mostrar las dos actividades: Invoke Ingestion e Invoke Transformation.  
Guardar la imagen como:

```text
docs/images/18_main_pipeline.png
```

Pegar aquí:

![Pipeline principal](docs/images/18_main_pipeline.png)

📌 **IMAGEN 19 - EJECUCIÓN EXITOSA DEL PIPELINE PRINCIPAL**  
Esta imagen debe mostrar el pipeline principal ejecutado correctamente.  
Guardar la imagen como:

```text
docs/images/19_main_pipeline_success.png
```

Pegar aquí:

![Ejecución exitosa del pipeline principal](docs/images/19_main_pipeline_success.png)

---

## Manejo de errores

El proyecto incluye un manejo básico de errores dentro del pipeline de ingesta.

El pipeline valida si existe la carpeta esperada antes de ejecutar los notebooks.

Si la carpeta no existe, se envía un correo electrónico de notificación.

### Escenario de error controlado

```text
Carpeta no encontrada
        |
        v
No ejecutar notebooks
        |
        v
Enviar email de error
```

Esto evita ejecutar procesos sobre datos inexistentes y permite notificar el problema.

📌 **IMAGEN 20 - ACTIVIDAD DE EMAIL ERROR**  
Esta imagen debe mostrar la actividad que envía el correo cuando falla la validación.  
Guardar la imagen como:

```text
docs/images/20_email_error_activity.png
```

Pegar aquí:

![Actividad de email error](docs/images/20_email_error_activity.png)

---

## Modelo semántico

Después de generar la capa Gold, se creó un **Semantic Model** en Microsoft Fabric.

El modelo semántico permite organizar las tablas, relaciones y métricas para su consumo desde Power BI.

### Tablas incluidas en el modelo

El modelo incluye tablas de dimensión y una tabla de hechos.

Ejemplo:

```text
fact_device
dim_category
dim_model
dim_brand
dim_camera
dim_display
dim_operating_system
dim_physical_specs
dim_time
```

### Diseño del modelo

El modelo tiene una estructura tipo estrella, donde `fact_device` funciona como tabla central y se relaciona con las tablas de dimensión.

📌 **IMAGEN 21 - MODELO SEMÁNTICO CON RELACIONES**  
Esta imagen debe ser la captura donde aparece `fact_device` al centro y las dimensiones alrededor.  
Guardar la imagen como:

```text
docs/images/21_semantic_model_relationships.png
```

Pegar aquí:

![Modelo semántico con relaciones](docs/images/21_semantic_model_relationships.png)

---

## Power BI Report y Dashboard

Con el modelo semántico se construyó un reporte en Power BI para analizar características de los dispositivos inteligentes.

El reporte permite visualizar métricas relacionadas con:

- Resolución de video.
- Píxeles efectivos de cámara.
- Tasa de refresco.
- Meses de lanzamiento.
- Cantidad de dispositivos.

📌 **IMAGEN 22 - REPORTE DE POWER BI**  
Esta imagen debe mostrar el reporte con visualizaciones de barras.  
Guardar la imagen como:

```text
docs/images/22_powerbi_report.png
```

Pegar aquí:

![Reporte de Power BI](docs/images/22_powerbi_report.png)

📌 **IMAGEN 23 - DASHBOARD DE POWER BI**  
Esta imagen debe mostrar el dashboard publicado en Microsoft Fabric.  
Guardar la imagen como:

```text
docs/images/23_powerbi_dashboard.png
```

Pegar aquí:

![Dashboard de Power BI](docs/images/23_powerbi_dashboard.png)

---

## Task Flow

Se utilizó **Task Flow** para organizar visualmente los elementos del proyecto dentro del workspace de Microsoft Fabric.

El Task Flow permite representar el proceso desde los datos de origen hasta la visualización final.

### Etapas representadas en el Task Flow

- Datos de bronze.
- Proceso de ingesta.
- Datos plateados.
- Transformación adicional.
- Datos de oro.
- Warehouse.
- Visualización de datos.
- Proceso Smart Data.

📌 **IMAGEN 24 - TASK FLOW DEL PROYECTO**  
Esta imagen debe ser la captura donde aparecen los bloques: Datos de bronce, Proceso Ingesta, Datos plateados, Transformación adicional, Datos de oro, Visualización de datos y Warehouse.  
Guardar la imagen como:

```text
docs/images/24_task_flow.png
```

Pegar aquí:

![Task Flow del proyecto](docs/images/24_task_flow.png)

---

## Lineage View

Se utilizó **Lineage View** para visualizar las dependencias entre los diferentes elementos del proyecto.

Esta vista permite entender cómo se conectan los datos desde el origen hasta los reportes finales.

### Beneficios del Lineage View

- Visualizar dependencias entre Lakehouses, pipelines, modelos y reportes.
- Entender el recorrido de los datos.
- Identificar impacto ante cambios.
- Documentar el flujo completo de la solución.

📌 **IMAGEN 25 - LINEAGE VIEW DEL PROYECTO**  
Esta imagen debe mostrar las dependencias entre los elementos del workspace.  
Guardar la imagen como:

```text
docs/images/25_lineage_view.png
```

Pegar aquí:

![Lineage View](docs/images/25_lineage_view.png)

---

## Deployment Pipelines

Se utilizó **Deployment Pipelines** para practicar el despliegue de elementos entre diferentes workspaces.

El flujo utilizado fue:

```text
Development → Test → Production
```

Esto permite simular un ciclo de vida más profesional para proyectos de datos y analítica.

### Workspaces utilizados

| Etapa | Descripción |
|---|---|
| Development | Workspace de desarrollo |
| Test | Workspace de pruebas |
| Production | Workspace de producción |

📌 **IMAGEN 26 - DEPLOYMENT PIPELINE DEV TEST PROD**  
Esta imagen debe mostrar las tres etapas: Development, Test y Production, con implementación correcta.  
Guardar la imagen como:

```text
docs/images/26_deployment_pipeline.png
```

Pegar aquí:

![Deployment Pipeline](docs/images/26_deployment_pipeline.png)

---

## Automatización

El proceso fue automatizado con pipelines de Microsoft Fabric.

La automatización permite ejecutar el flujo completo sin correr manualmente cada notebook o dataflow.

### Automatización implementada

- Pipeline de ingesta.
- Pipeline de transformación.
- Pipeline principal.
- Paso de parámetros entre pipelines.
- Validación previa de carpetas.
- Notificación por correo ante error.
- Programación del pipeline.

Parámetros principales:

```text
file_date
environment
```

---

## Complicaciones encontradas y soluciones

Durante el desarrollo del proyecto surgieron varios problemas técnicos. Estos errores fueron resueltos durante la implementación.

Esta sección documenta las principales complicaciones y cómo fueron solucionadas.

---

### 1. Parametrización de notebooks

**Problema**

Algunos notebooks usaban valores fijos para la fecha de procesamiento y el ambiente.

Esto hacía que el proceso no fuera reutilizable desde pipelines.

Ejemplo del problema:

```text
Ruta fija con fecha escrita manualmente
```

**Solución**

Se agregaron parámetros reutilizables:

```text
file_date
environment
```

Estos parámetros fueron enviados desde los pipelines hacia los notebooks.

Resultado:

```text
El mismo notebook puede ejecutarse para diferentes fechas y ambientes.
```

---

### 2. Error en función reutilizable

**Problema**

Varios notebooks fallaban con el siguiente error:

```text
TypeError: add_ingestion_date() missing 1 required positional argument: 'file_date'
```

**Causa**

La función `add_ingestion_date()` esperaba recibir el parámetro `file_date`, pero algunos notebooks no lo estaban enviando.

**Solución**

Se corrigieron las llamadas a la función en los notebooks de ingesta para enviar correctamente el parámetro.

Ejemplo conceptual:

```python
df = add_ingestion_date(df, file_date)
```

Resultado:

```text
Los notebooks de ingesta pudieron ejecutarse correctamente.
```

---

### 3. Error de esquema en Delta Lake

**Problema**

Al escribir datos en tablas Delta apareció un error de diferencia de esquema.

```text
AnalysisException: A schema mismatch detected when writing to the Delta table
```

**Causa**

Algunas tablas Delta ya existían con un esquema diferente al DataFrame que se intentaba guardar.

Por ejemplo, algunas tablas no esperaban columnas como:

```text
file_date
envrionment
environment
```

**Solución**

Se ajustaron los notebooks para alinear el DataFrame con el esquema esperado.

En algunos casos se eliminaron columnas innecesarias antes de escribir.

En otros casos se utilizó una escritura con actualización de esquema cuando era necesario.

Resultado:

```text
Las tablas Delta pudieron escribirse correctamente.
```

---

### 4. Variable no definida en notebook de transformación

**Problema**

Uno de los notebooks de transformación falló con el siguiente error:

```text
NameError: name 'device_final_df' is not defined
```

**Causa**

Se estaba utilizando una variable que no había sido creada con ese nombre.

**Solución**

Se corrigió el nombre del DataFrame y se definió correctamente antes de escribir la tabla final.

Resultado:

```text
La transformación de Device pudo ejecutarse correctamente.
```

---

### 5. Error al importar notebooks en Microsoft Fabric

**Problema**

Microsoft Fabric devolvía un error al importar algunos notebooks:

```text
400 Bad Request
```

**Causa**

Algunos notebooks contenían metadatos incompatibles con el importador de Microsoft Fabric.

**Solución**

Se recrearon versiones limpias de los notebooks, eliminando metadatos incompatibles.

Resultado:

```text
Los notebooks pudieron importarse correctamente en Microsoft Fabric.
```

---

### 6. Límite de capacidad Spark

**Problema**

Durante la ejecución de varios notebooks apareció un error de capacidad:

```text
TooManyRequestsForCapacity
```

**Causa**

Se estaban ejecutando varias sesiones Spark al mismo tiempo dentro de una capacidad Trial de Microsoft Fabric.

**Solución**

Se controló mejor la ejecución de notebooks y pipelines para evitar exceso de concurrencia.

Resultado:

```text
Los procesos pudieron ejecutarse sin saturar la capacidad disponible.
```

---

### 7. Error en Dataflows Gen2

**Problema**

Algunos Dataflows fallaban durante la transformación.

**Causa**

Existían pasos que intentaban eliminar columnas que no estaban presentes en el esquema actual.

Ejemplo:

```text
Eliminar columna envrionment
```

**Solución**

Se revisaron los pasos de transformación en Power Query y se corrigieron para trabajar con las columnas correctas.

Resultado:

```text
Los Dataflows Gen2 pudieron ejecutarse correctamente.
```

---

### 8. Paso de parámetros entre pipelines

**Problema**

El pipeline principal necesitaba invocar los pipelines hijos manteniendo los mismos parámetros de ejecución.

**Solución**

Se configuraron los parámetros:

```text
file_date
environment
```

en el pipeline principal y se enviaron hacia:

```text
pl_ingest_smart_device
pl_transformation_smart_device
```

Resultado:

```text
El flujo completo pudo ejecutarse de forma parametrizada.
```

---

## Resultados obtenidos

Al finalizar el proyecto se logró construir una solución completa de datos en Microsoft Fabric.

Resultados principales:

- Dataset cargado desde Kaggle hacia Azure Data Lake Storage Gen2.
- Archivos cargados mediante Azure Storage Explorer.
- Datos organizados por fecha de procesamiento.
- Conexión con Fabric mediante OneLake Shortcut.
- Lakehouse organizado en capas Bronze, Silver y Gold.
- Notebooks de ingesta desarrollados con PySpark.
- Dataflows Gen2 creados para transformaciones.
- Notebooks de transformación creados para la capa Gold.
- Cargas completas e incrementales implementadas.
- Pipelines de ingesta y transformación automatizados.
- Pipeline principal ejecutando el flujo completo.
- Validación previa de carpetas.
- Notificación por correo en caso de error.
- Modelo semántico creado.
- Reporte de Power BI creado.
- Dashboard creado.
- Task Flow configurado.
- Lineage View revisado.
- Deployment Pipeline aplicado entre Development, Test y Production.

📌 **IMAGEN 27 - TODOS LOS PROCESOS EN ESTADO CORRECTO**  
Esta imagen debe mostrar el pipeline o las actividades completas en verde/correcto.  
Guardar la imagen como:

```text
docs/images/27_all_processes_success.png
```

Pegar aquí:

![Procesos ejecutados correctamente](docs/images/27_all_processes_success.png)

---

## Aprendizajes principales

Este proyecto permitió practicar conceptos importantes de ingeniería de datos y analítica moderna.

Aprendizajes principales:

- Diseño de arquitectura Medallion.
- Uso de Lakehouse en Microsoft Fabric.
- Integración entre Azure Data Lake Storage Gen2 y OneLake.
- Uso de Azure Storage Explorer para carga de archivos.
- Uso de Shortcuts en OneLake.
- Procesamiento de datos con PySpark.
- Uso de Spark DataFrames.
- Escritura de tablas Delta.
- Creación de notebooks parametrizados.
- Reutilización de código entre notebooks.
- Implementación de cargas completas.
- Implementación de cargas incrementales.
- Transformación de datos con Dataflows Gen2.
- Orquestación con Data Factory Pipelines.
- Manejo básico de errores.
- Envío de notificaciones por correo.
- Creación de modelo semántico.
- Creación de reporte y dashboard en Power BI.
- Uso de Task Flow para organizar el proyecto.
- Uso de Lineage View para visualizar dependencias.
- Uso de Deployment Pipelines para simular ambientes Dev, Test y Prod.

---

## Mejoras futuras

Algunas mejoras que podrían agregarse en una siguiente versión:

- Implementar Real-Time Intelligence con Eventstream y Eventhouse.
- Agregar validaciones de calidad de datos.
- Agregar pruebas automáticas para notebooks.
- Crear reglas de negocio documentadas por tabla.
- Mejorar el manejo de errores.
- Agregar logs técnicos de ejecución.
- Crear métricas DAX adicionales.
- Mejorar el diseño visual del reporte en Power BI.
- Separar de forma más completa los ambientes Development, Test y Production.
- Automatizar el despliegue de artefactos.
- Implementar monitoreo más detallado de pipelines.

---

## Estado del proyecto

Estado actual:

```text
Completado
```

Componentes implementados:

| Componente | Estado |
|---|---|
| Azure Data Lake Storage Gen2 | Completado |
| Azure Storage Explorer | Completado |
| OneLake Shortcut | Completado |
| Bronze Layer | Completado |
| Silver Layer | Completado |
| Gold Layer | Completado |
| Ingestion Notebooks | Completado |
| Transformation Notebooks | Completado |
| Dataflows Gen2 | Completado |
| Pipelines | Completado |
| Email Error Notification | Completado |
| Semantic Model | Completado |
| Power BI Report | Completado |
| Dashboard | Completado |
| Task Flow | Completado |
| Lineage View | Completado |
| Deployment Pipelines | Completado |
| Real-Time Intelligence | No implementado en esta versión |

---

## Nota sobre seguridad

Este repositorio no incluye credenciales, claves de acceso, cadenas de conexión ni información sensible.

Antes de subir capturas al repositorio, se recomienda ocultar:

- Correo personal.
- Subscription ID.
- Tenant ID.
- Nombres internos sensibles.
- Claves de acceso.
- URLs privadas.
- Identificadores internos del workspace.

---

## Autor

Desarrollado por: **TU NOMBRE AQUÍ**

LinkedIn: **TU LINKEDIN AQUÍ**

GitHub: **TU GITHUB AQUÍ**
