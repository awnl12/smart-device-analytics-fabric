# Smart Device Analytics Platform con Microsoft Fabric

Proyecto end-to-end de **ingeniería de datos y analítica** desarrollado con **Microsoft Fabric**, implementando una arquitectura tipo **Medallion Architecture** con capas **Bronze, Silver y Gold**.

El proyecto utiliza un dataset de dispositivos inteligentes con información histórica sobre smartphones, tablets, wearables y otros dispositivos. La solución permite cargar, transformar, modelar, orquestar y visualizar datos utilizando herramientas modernas del ecosistema de Microsoft Fabric.

Este proyecto fue desarrollado con fines académicos y de práctica profesional, complementando la implementación con resolución de errores reales, parametrización de notebooks, corrección de esquemas, manejo básico de errores, orquestación de pipelines y prácticas de deployment entre workspaces.

---

## Tabla de contenidos

- [Objetivo del proyecto](#objetivo-del-proyecto)
- [Dataset utilizado](#dataset-utilizado)
- [Arquitectura de la solución](#arquitectura-de-la-solución)
- [Tecnologías utilizadas](#tecnologías-utilizadas)
- [Estructura del proyecto en Microsoft Fabric](#estructura-del-proyecto-en-microsoft-fabric)
- [Flujo general del proyecto](#flujo-general-del-proyecto)
- [Capa Bronze](#capa-bronze)
- [Capa Silver](#capa-silver)
- [Capa Gold](#capa-gold)
- [Notebooks](#notebooks)
- [Dataflows Gen2](#dataflows-gen2)
- [Copy Job](#copy-job)
- [Carga completa e incremental](#carga-completa-e-incremental)
- [Orquestación con Pipelines](#orquestación-con-pipelines)
- [Manejo de errores](#manejo-de-errores)
- [Warehouse](#warehouse)
- [Modelo semántico](#modelo-semántico)
- [Power BI Report y Dashboard](#power-bi-report-y-dashboard)
- [Task Flow](#task-flow)
- [Lineage View](#lineage-view)
- [Deployment Pipelines](#deployment-pipelines)
- [Complicaciones encontradas y soluciones](#complicaciones-encontradas-y-soluciones)
- [Resultados obtenidos](#resultados-obtenidos)
- [Aprendizajes principales](#aprendizajes-principales)
- [Mejoras futuras](#mejoras-futuras)
- [Estado del proyecto](#estado-del-proyecto)
- [Nota sobre seguridad](#nota-sobre-seguridad)
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
- Uso de Copy Job como alternativa de carga dentro de Fabric.
- Implementación de cargas completas e incrementales.
- Orquestación con Data Factory Pipelines.
- Parametrización de procesos con `file_date` y `environment`.
- Validación de carpetas antes de ejecutar la ingesta.
- Notificación por correo electrónico en caso de error.
- Creación de Warehouse.
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

Ejemplo de estructura en el Data Lake:

```text
smart-device/
└── bronze-data/
    ├── 2025-07-07/
    ├── 2025-07-14/
    └── 2025-07-21/
```

Posteriormente, esta ubicación fue conectada con Microsoft Fabric mediante un **OneLake Shortcut**, permitiendo que el Lakehouse Bronze acceda a los archivos almacenados en ADLS Gen2.

---

### Evidencia del dataset y carga inicial

📌 **IMAGEN 01 - FUENTE DEL DATASET EN KAGGLE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de Kaggle mostrando el dataset Smart Device.
- Debe verse el nombre del dataset o la página de origen.

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 02 - AZURE STORAGE EXPLORER CON LOS ARCHIVOS CARGADOS**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de Azure Storage Explorer.
- Deben verse los archivos del dataset cargados hacia el contenedor o carpeta del proyecto.
- Esta imagen demuestra que el dataset fue cargado manualmente hacia ADLS Gen2.

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 03 - CONTENEDOR EN AZURE DATA LAKE STORAGE GEN2 CON CARPETAS POR FECHA**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del contenedor `smart-device`.
- Debe verse la carpeta `bronze-data`.
- Deben verse carpetas como:
  - `2025-07-07`
  - `2025-07-14`
  - `2025-07-21`

**PEGAR IMAGEN AQUÍ**

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

Descripción exacta de la imagen que debes pegar aquí:

- Diagrama propio que muestre el flujo completo.
- Debe verse algo parecido a:
  - Kaggle / ADLS Gen2
  - OneLake Shortcut
  - Bronze
  - Silver
  - Gold
  - Warehouse
  - Semantic Model
  - Power BI

**PEGAR IMAGEN AQUÍ**

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
| Warehouse | Almacenamiento analítico para consumo de datos |
| Data Factory Pipelines | Orquestación de ingesta, transformación y reportes |
| Fabric Notebooks | Desarrollo de procesos de ingesta y transformación |
| PySpark | Procesamiento y transformación de datos |
| Spark DataFrames | Lectura, limpieza y transformación de datos |
| Delta Lake | Escritura de tablas Delta |
| Dataflows Gen2 | Transformaciones visuales con Power Query |
| Power Query | Transformación de datos en Dataflows Gen2 |
| Copy Job | Carga/copia de datos como alternativa en Fabric |
| Semantic Model | Modelo analítico para Power BI |
| Power BI | Reportes y dashboard |
| Task Flow | Organización visual del proyecto |
| Lineage View | Visualización de dependencias |
| Deployment Pipelines | Despliegue entre workspaces |

---

## Estructura del proyecto en Microsoft Fabric

El workspace fue organizado en carpetas para separar los componentes principales del proyecto.

```text
Smart_Device_Analytics_WS/
│
├── copy_job/
│   └── cj_smart_device
│
├── dataflow gen2/
│   ├── 01-Transformation_table_brand
│   ├── 02-Transformation_table_camera
│   ├── 03-Transformation_table_connectivity
│   └── 04-Transformation_table_operating_system
│
├── notebook/
│   ├── email/
│   │   └── email ERROR
│   │
│   ├── includes/
│   │   ├── common_functions
│   │   └── configuration
│   │
│   ├── ingestion/
│   │   ├── 02.Ingestion_file_model
│   │   ├── 03.Ingestion_file_category
│   │   ├── 04.Ingestion_file_camera
│   │   ├── 05.Ingestion_file_brand
│   │   ├── 06.Ingestion_file_operationg_system
│   │   ├── 07.Ingestion_file_connectivity
│   │   ├── 08.Ingestion_file_display
│   │   └── 09.Ingestion_file_physical_specs
│   │
│   └── transformation/
│       ├── 01.Transformation_table_category
│       ├── 02.Transformation_table_device
│       ├── 03.Transformation_table_model
│       ├── 04.Transformation_table_display
│       ├── 05.Transformation_table_physical_specs
│       └── 06.Create_dim_time
│
├── pipeline/
│   ├── pl_ingest_smart_device
│   ├── pl_process_smart_device
│   ├── pl_report_smart_data
│   └── pl_transformation_smart_device
│
├── report/
│   ├── report_device
│   └── dashboard_smart_device
│
├── lh_bronze
├── lh_silver
├── lh_gold
├── wh_smart_device
└── sm_smart_device_wh
```

> Nota: El elemento `06.Ingestion_file_operationg_system` conserva el nombre utilizado dentro del proyecto, aunque contiene un error tipográfico en la palabra `operationg`.

📌 **IMAGEN 05 - ESTRUCTURA GENERAL DEL WORKSPACE EN MICROSOFT FABRIC**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del workspace donde se vean las carpetas:
  - `copy_job`
  - `dataflow gen2`
  - `notebook`
  - `pipeline`
  - `report`
- También deben verse elementos como:
  - `lh_bronze`
  - `lh_silver`
  - `lh_gold`
  - `wh_smart_device`
  - `sm_smart_device_wh`

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 06 - CARPETA NOTEBOOK CON SUBCARPETAS**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de la carpeta `notebook`.
- Deben verse las carpetas:
  - `email`
  - `includes`
  - `ingestion`
  - `transformation`

**PEGAR IMAGEN AQUÍ**

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

📌 **IMAGEN 07 - RESOURCE GROUP DE AZURE DEL PROYECTO**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del grupo de recursos de Azure.
- Debe verse la cuenta de almacenamiento usada para el proyecto.
- Antes de subirla, ocultar:
  - correo
  - subscription ID
  - tenant ID
  - datos personales

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 08 - CARPETAS DE BRONZE EN AZURE DATA LAKE STORAGE GEN2**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del contenedor `smart-device`.
- Debe verse:
  - `bronze-data`
  - carpetas por fecha
  - archivos o estructura del dataset

**PEGAR IMAGEN AQUÍ**

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
| Model | Información de modelos |
| Category | Categorías de dispositivos |
| Camera | Características de cámara |
| Brand | Información de marcas |
| Operating System | Sistemas operativos |
| Connectivity | Información de conectividad |
| Display | Características de pantalla |
| Physical Specs | Especificaciones físicas |

📌 **IMAGEN 09 - LAKEHOUSE SILVER CON TABLAS GENERADAS**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del Lakehouse `lh_silver`.
- Deben verse las tablas generadas por los notebooks de ingesta.

**PEGAR IMAGEN AQUÍ**

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

📌 **IMAGEN 10 - LAKEHOUSE GOLD CON TABLAS FINALES**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de `lh_gold`.
- Deben verse tablas como:
  - `fact_device`
  - `dim_category`
  - `dim_model`
  - `dim_brand`
  - `dim_camera`
  - `dim_display`
  - `dim_operating_system`
  - `dim_physical_specs`
  - `dim_time`

**PEGAR IMAGEN AQUÍ**

---

## Notebooks

Los notebooks fueron organizados en cuatro grupos:

1. Notebooks de configuración e includes.
2. Notebook de email.
3. Notebooks de ingesta.
4. Notebooks de transformación.

---

### 1. Notebooks de configuración e includes

| Notebook | Uso |
|---|---|
| configuration | Centraliza parámetros o rutas utilizadas por el proyecto |
| common_functions | Contiene funciones reutilizables para ingesta y transformación |

Estos notebooks permiten reutilizar código y evitar repetir lógica en cada notebook de ingesta o transformación.

📌 **IMAGEN 11 - CARPETA INCLUDES CON CONFIGURATION Y COMMON_FUNCTIONS**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de `notebook/includes`.
- Deben verse:
  - `configuration`
  - `common_functions`

**PEGAR IMAGEN AQUÍ**

---

### 2. Notebook de email

| Notebook | Uso |
|---|---|
| email ERROR | Envía notificación por correo cuando ocurre un error controlado |

Este notebook se utiliza desde el pipeline de ingesta cuando la validación de carpeta falla.

📌 **IMAGEN 12 - NOTEBOOK EMAIL ERROR**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del notebook `email ERROR`.
- Debe verse que está relacionado con el envío de correo o notificación de error.

**PEGAR IMAGEN AQUÍ**

---

### 3. Notebooks de ingesta

Los notebooks de ingesta leen datos desde la capa Bronze y escriben tablas en la capa Silver.

| Notebook | Descripción |
|---|---|
| 02.Ingestion_file_model | Ingesta de modelos |
| 03.Ingestion_file_category | Ingesta de categorías |
| 04.Ingestion_file_camera | Ingesta de cámaras |
| 05.Ingestion_file_brand | Ingesta de marcas |
| 06.Ingestion_file_operationg_system | Ingesta de sistemas operativos |
| 07.Ingestion_file_connectivity | Ingesta de conectividad |
| 08.Ingestion_file_display | Ingesta de pantallas |
| 09.Ingestion_file_physical_specs | Ingesta de especificaciones físicas |

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

### Parámetros usados en notebooks de ingesta

```text
file_date
environment
```

Ejemplo:

```text
file_date = 2025-07-07
environment = Development
```

📌 **IMAGEN 13 - CARPETA INGESTION CON NOTEBOOKS DE INGESTA**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de `notebook/ingestion`.
- Deben verse los notebooks de ingesta listados.

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 14 - NOTEBOOK DE INGESTA CON CÓDIGO PYSPARK**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de un notebook de ingesta abierto.
- Debe verse código PySpark leyendo datos, aplicando schema o escribiendo en Delta.
- No es necesario mostrar todo el notebook, solo una parte representativa.

**PEGAR IMAGEN AQUÍ**

---

### 4. Notebooks de transformación

Los notebooks de transformación leen datos desde Silver y generan tablas finales en Gold.

| Notebook | Descripción |
|---|---|
| 01.Transformation_table_category | Transformación de categoría |
| 02.Transformation_table_device | Transformación de dispositivo |
| 03.Transformation_table_model | Transformación de modelo |
| 04.Transformation_table_display | Transformación de pantalla |
| 05.Transformation_table_physical_specs | Transformación de especificaciones físicas |
| 06.Create_dim_time | Creación de dimensión de tiempo |

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

📌 **IMAGEN 15 - CARPETA TRANSFORMATION CON NOTEBOOKS DE TRANSFORMACIÓN**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de `notebook/transformation`.
- Deben verse los notebooks de transformación.

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 16 - NOTEBOOK DE TRANSFORMACIÓN CON CÓDIGO PYSPARK**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de un notebook de transformación abierto.
- Puede ser:
  - `02.Transformation_table_device`
  - `03.Transformation_table_model`
  - `06.Create_dim_time`
- Debe verse lógica de transformación, merge, escritura o creación de dimensión.

**PEGAR IMAGEN AQUÍ**

---

## Dataflows Gen2

Se utilizaron **Dataflows Gen2** para transformar algunas tablas de referencia de forma visual usando Power Query.

Los Dataflows creados fueron:

| Dataflow Gen2 | Objetivo |
|---|---|
| 01-Transformation_table_brand | Transformar datos de marcas |
| 02-Transformation_table_camera | Transformar datos de cámaras |
| 03-Transformation_table_connectivity | Transformar datos de conectividad |
| 04-Transformation_table_operating_system | Transformar datos de sistemas operativos |

Estos Dataflows forman parte del pipeline de transformación y cargan datos hacia la capa Gold.

📌 **IMAGEN 17 - CARPETA DATAFLOW GEN2 CON LOS 4 DATAFLOWS**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de `dataflow gen2`.
- Deben verse:
  - `01-Transformation_table_brand`
  - `02-Transformation_table_camera`
  - `03-Transformation_table_connectivity`
  - `04-Transformation_table_operating_system`

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 18 - EJEMPLO DE DATAFLOW GEN2 ABIERTO**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de un Dataflow Gen2 abierto.
- Debe verse Power Query o el flujo de transformación.
- Puede ser cualquiera de:
  - Brand
  - Camera
  - Connectivity
  - Operating System

**PEGAR IMAGEN AQUÍ**

---

## Copy Job

Además del flujo principal con notebooks, Dataflows Gen2 y pipelines, también se practicó el uso de un elemento de tipo **Copy Job** en Microsoft Fabric.

El elemento creado fue:

```text
cj_smart_device
```

Este componente permitió explorar una alternativa para copiar datos dentro del ecosistema de Fabric y comparar sus limitaciones frente a pipelines y notebooks.

📌 **IMAGEN 19 - COPY JOB CJ_SMART_DEVICE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de la carpeta `copy_job`.
- Debe verse el elemento `cj_smart_device`.
- También puede ser una captura del Copy Job abierto.

**PEGAR IMAGEN AQUÍ**

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

Los pipelines creados fueron:

| Pipeline | Descripción |
|---|---|
| pl_ingest_smart_device | Ejecuta el proceso de ingesta desde Bronze hacia Silver |
| pl_transformation_smart_device | Ejecuta transformaciones desde Silver hacia Gold |
| pl_process_smart_device | Pipeline principal que invoca ingesta y transformación |
| pl_report_smart_data | Pipeline relacionado con automatización del proceso de reporte |

📌 **IMAGEN 20 - CARPETA PIPELINE CON LAS CANALIZACIONES CREADAS**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de `pipeline`.
- Deben verse:
  - `pl_ingest_smart_device`
  - `pl_process_smart_device`
  - `pl_report_smart_data`
  - `pl_transformation_smart_device`

**PEGAR IMAGEN AQUÍ**

---

### Pipeline de ingesta

El pipeline `pl_ingest_smart_device` valida primero si existe la carpeta de datos correspondiente a la fecha de procesamiento.

Flujo:

```text
Get Folder Details
        |
        v
If Condition
        |
        |-- True  → Ejecutar notebooks de ingesta
        |
        |-- False → Ejecutar notebook email ERROR
```

Si la carpeta existe, se ejecutan los notebooks de ingesta.

Si la carpeta no existe, se ejecuta la notificación de error.

📌 **IMAGEN 21 - PL_INGEST_SMART_DEVICE ABIERTO**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del pipeline de ingesta abierto.
- Deben verse:
  - `Get Folder Details`
  - `If Condition`
  - Notebooks de ingesta
  - Ruta de error

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 22 - IF CONDITION CON RUTA TRUE Y RUTA FALSE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura dentro de la condición IF.
- Debe verse:
  - En `True`: actividades de ingesta.
  - En `False`: `emailError` o notebook de email.

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 23 - PL_INGEST_SMART_DEVICE EJECUTADO CORRECTAMENTE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de la salida del pipeline de ingesta.
- Deben verse las actividades en estado:
  - Correcto
  - Verde
  - Successful

**PEGAR IMAGEN AQUÍ**

---

### Pipeline de transformación

El pipeline `pl_transformation_smart_device` ejecuta notebooks y Dataflows Gen2 para crear la capa Gold.

Procesos dentro del pipeline:

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

📌 **IMAGEN 24 - PL_TRANSFORMATION_SMART_DEVICE ABIERTO**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del pipeline de transformación.
- Deben verse notebooks y Dataflows Gen2.

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 25 - PL_TRANSFORMATION_SMART_DEVICE EJECUTADO CORRECTAMENTE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de la salida del pipeline de transformación.
- Deben verse actividades en verde o con estado `Correcto`.

**PEGAR IMAGEN AQUÍ**

---

### Pipeline principal

El pipeline `pl_process_smart_device` invoca primero la ingesta y luego la transformación.

Flujo:

```text
Invoke Ingestion
        |
        v
Invoke Transformation
```

Este pipeline permite ejecutar el proceso completo de forma automatizada.

Parámetros principales:

```text
file_date
environment
```

📌 **IMAGEN 26 - PL_PROCESS_SMART_DEVICE CON INVOKE INGESTION E INVOKE TRANSFORMATION**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del pipeline principal.
- Deben verse dos actividades:
  - `Invoke Ingestion`
  - `Invoke Transformation`

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 27 - PL_PROCESS_SMART_DEVICE EJECUTADO CORRECTAMENTE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del pipeline principal en ejecución exitosa.
- Debe verse el estado de la canalización en correcto.

**PEGAR IMAGEN AQUÍ**

---

### Pipeline de reporte

El pipeline `pl_report_smart_data` fue utilizado para automatizar el proceso relacionado con la capa de análisis y reporte.

Este pipeline complementa el flujo principal, permitiendo integrar el procesamiento de datos con los elementos analíticos del proyecto.

📌 **IMAGEN 28 - PL_REPORT_SMART_DATA**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del pipeline `pl_report_smart_data`.
- Debe verse la actividad o flujo relacionado con el reporte.

**PEGAR IMAGEN AQUÍ**

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

📌 **IMAGEN 29 - ACTIVIDAD O NOTEBOOK DE EMAIL ERROR**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del notebook `email ERROR` o de la actividad del pipeline que lo ejecuta.
- Debe quedar claro que el flujo envía un correo cuando falla la validación.

**PEGAR IMAGEN AQUÍ**

---

## Warehouse

Además de los Lakehouses, el proyecto incluye un Warehouse:

```text
wh_smart_device
```

El Warehouse fue utilizado como parte de la capa analítica del proyecto y se integra con el modelo semántico y Power BI.

📌 **IMAGEN 30 - WAREHOUSE WH_SMART_DEVICE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del elemento `wh_smart_device`.
- Puede ser la vista del Warehouse o sus tablas.

**PEGAR IMAGEN AQUÍ**

---

## Modelo semántico

Después de generar la capa Gold, se creó un **Semantic Model** en Microsoft Fabric.

El modelo semántico permite organizar las tablas, relaciones y métricas para su consumo desde Power BI.

Elemento creado:

```text
sm_smart_device_wh
```

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

📌 **IMAGEN 31 - MODELO SEMÁNTICO CON RELACIONES**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del modelo semántico.
- Debe verse:
  - `fact_device` en el centro.
  - Dimensiones alrededor.
  - Relaciones entre tablas.

**PEGAR IMAGEN AQUÍ**

---

## Power BI Report y Dashboard

Con el modelo semántico se construyó un reporte en Power BI para analizar características de los dispositivos inteligentes.

Elementos creados:

| Elemento | Descripción |
|---|---|
| report_device | Reporte de análisis de dispositivos |
| dashboard_smart_device | Dashboard publicado a partir del reporte |

El reporte permite analizar características como:

- Resolución de video.
- Píxeles efectivos de cámara.
- Tasa de refresco.
- Meses de lanzamiento.
- Cantidad de dispositivos.

📌 **IMAGEN 32 - REPORTE REPORT_DEVICE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del reporte `report_device`.
- Deben verse gráficos de análisis.
- Por ejemplo:
  - video recording
  - effective pixel
  - refresh rate
  - month name

**PEGAR IMAGEN AQUÍ**

---

📌 **IMAGEN 33 - DASHBOARD DASHBOARD_SMART_DEVICE**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del dashboard `dashboard_smart_device`.
- Deben verse visualizaciones publicadas en el dashboard.

**PEGAR IMAGEN AQUÍ**

---

## Task Flow

Se utilizó **Task Flow** para organizar visualmente los elementos del proyecto dentro del workspace de Microsoft Fabric.

El Task Flow permite representar el proceso desde los datos de origen hasta la visualización final.

### Etapas representadas en el Task Flow

- Datos de bronce.
- Proceso Ingesta.
- Datos plateados.
- Transformación adicional.
- Datos de oro.
- Warehouse.
- Visualización de datos.
- Process Smart Data.

📌 **IMAGEN 34 - TASK FLOW DEL PROYECTO**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del Task Flow.
- Deben verse bloques como:
  - `Datos de bronce`
  - `Proceso Ingesta`
  - `Datos plateados`
  - `Transformación adicional`
  - `Datos de oro`
  - `Warehouse`
  - `Visualización de datos`
  - `Process Smart Data`

**PEGAR IMAGEN AQUÍ**

---

## Lineage View

Se utilizó **Lineage View** para visualizar las dependencias entre los diferentes elementos del proyecto.

Esta vista permite entender cómo se conectan los datos desde el origen hasta los reportes finales.

### Beneficios del Lineage View

- Visualizar dependencias entre Lakehouses, pipelines, modelos y reportes.
- Entender el recorrido de los datos.
- Identificar impacto ante cambios.
- Documentar el flujo completo de la solución.

📌 **IMAGEN 35 - LINEAGE VIEW DEL PROYECTO**

Descripción exacta de la imagen que debes pegar aquí:

- Captura de Lineage View.
- Deben verse conexiones entre:
  - Lakehouses
  - Pipelines
  - Warehouse
  - Semantic Model
  - Report
  - Dashboard

**PEGAR IMAGEN AQUÍ**

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

📌 **IMAGEN 36 - DEPLOYMENT PIPELINE DEV TEST PROD**

Descripción exacta de la imagen que debes pegar aquí:

- Captura del Deployment Pipeline.
- Deben verse las tres etapas:
  - Development
  - Test
  - Production
- Debe verse implementación correcta o exitosa.

**PEGAR IMAGEN AQUÍ**

---

## Automatización

El proceso fue automatizado con pipelines de Microsoft Fabric.

La automatización permite ejecutar el flujo completo sin correr manualmente cada notebook o dataflow.

### Automatización implementada

- Pipeline de ingesta.
- Pipeline de transformación.
- Pipeline principal.
- Pipeline de reporte.
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

**Resultado**

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

**Resultado**

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

**Resultado**

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

**Resultado**

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

**Resultado**

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

**Resultado**

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

**Resultado**

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

**Resultado**

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
- Notebooks de configuración y funciones reutilizables.
- Notebook de email para errores controlados.
- Dataflows Gen2 creados para transformaciones.
- Copy Job creado como práctica adicional.
- Notebooks de transformación creados para la capa Gold.
- Cargas completas e incrementales implementadas.
- Pipelines de ingesta y transformación automatizados.
- Pipeline principal ejecutando el flujo completo.
- Pipeline de reporte creado.
- Validación previa de carpetas.
- Notificación por correo en caso de error.
- Warehouse creado.
- Modelo semántico creado.
- Reporte de Power BI creado.
- Dashboard creado.
- Task Flow configurado.
- Lineage View revisado.
- Deployment Pipeline aplicado entre Development, Test y Production.

📌 **IMAGEN 37 - TODOS LOS PROCESOS EN ESTADO CORRECTO**

Descripción exacta de la imagen que debes pegar aquí:

- Captura donde se vea el estado correcto de las actividades.
- Puede ser del pipeline principal, ingesta o transformación.
- Idealmente debe mostrar varias actividades con check verde.

**PEGAR IMAGEN AQUÍ**

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
- Uso de Copy Job.
- Orquestación con Data Factory Pipelines.
- Manejo básico de errores.
- Envío de notificaciones por correo.
- Uso de Warehouse.
- Creación de modelo semántico.
- Creación de reporte y dashboard en Power BI.
- Uso de Task Flow para organizar el proyecto.
- Uso de Lineage View para visualizar dependencias.
- Uso de Deployment Pipelines para simular ambientes Dev, Test y Prod.

---

## Alcance del aprendizaje aplicado

Durante el desarrollo del proyecto se aplicaron conceptos de varias áreas de Microsoft Fabric:

### Fundamentos de Microsoft Fabric

- Workspaces.
- Workloads.
- Roles.
- Capacidad Trial.
- OneLake.
- Lakehouse.
- Warehouse.
- SQL Analytics Endpoint.

### Ingeniería de datos

- Apache Spark.
- PySpark.
- DataFrames.
- Notebooks.
- Schemas.
- Lectura y escritura de archivos.
- Tablas Delta.
- Reutilización de código.
- Parámetros en notebooks.
- Funciones reutilizables.

### Transformación de datos

- Dataflows Gen2.
- Power Query.
- Transformaciones con notebooks.
- Cargas completas.
- Cargas incrementales.
- Creación de dimensión de tiempo.
- Creación de dimensiones y tabla de hechos.

### Orquestación

- Pipelines.
- Actividades de pipeline.
- Validaciones.
- Condiciones IF.
- Invocación de pipelines.
- Programación.
- Notificaciones por correo.

### Analítica

- Warehouse.
- Semantic Model.
- Power BI Report.
- Dashboard.
- Visualizaciones.

### Gobierno y ciclo de vida

- Task Flow.
- Lineage View.
- Deployment Pipelines.
- Workspaces de Development, Test y Production.

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
| Includes / Common Functions | Completado |
| Email Error Notebook | Completado |
| Transformation Notebooks | Completado |
| Dataflows Gen2 | Completado |
| Copy Job | Completado |
| Pipelines | Completado |
| Pipeline principal | Completado |
| Pipeline de reporte | Completado |
| Email Error Notification | Completado |
| Warehouse | Completado |
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
- Información privada de Azure.
- Información privada de Microsoft Fabric.

---

## Autor

Desarrollado por: jORGE MARROQUIN RODRIGUEZ

LinkedIn: www.linkedin.com/in/marroquinrj

GitHub: awnl12
