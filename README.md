# Formula 1 Azure Databricks Pipeline

Este proyecto implementa una solución completa de Lakehouse sobre **Azure Databricks** para el procesamiento, transformación y análisis de más de 70 años de datos del campeonato de Formula 1 (1950 – Actualidad).

El sistema pasa de ingestar datos crudos en formatos heterogéneos (CSV y JSON anidados) a construir una capa analítica optimizada mediante la **Arquitectura Medallion** (Bronze, Silver, Gold) mediante el uso de **Apache Spark**. El almacenamiento de los datos se realiza en **Azure (ADLS Gen2)** (mediante una capa *landing* que contiene el dato en crudo y el uso de **Delta Tables**) y la orquestación del flujo de trabajo se gestiona mediante **Lakeflow Jobs**. Ofrece una infraestructura escalable con soporte para cargas incrementales, control de esquemas, linaje de datos mediante Unity Catalog y gestión de entorno simplificada.

![Azure Databricks](https://img.shields.io/badge/Azure_Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![Apache Spark](https://img.shields.io/badge/Apache_Spark-PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta_Lake-000000?style=for-the-badge&logo=delta&logoColor=white)
![Azure Data Lake](https://img.shields.io/badge/Azure_ADLS_Gen2-Storage-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Databricks Jobs](https://img.shields.io/badge/lakeflow_Jobs-Orchestration-FF3621?style=for-the-badge&logo=databricks&logoColor=white)

--- 

## Arquitectura

### 1. Diagrama End-to-End (Medallion)

```mermaid
graph LR
    subgraph " "
        GH_API["Landing (Inbound Data)"] -->|Raw_Ingestion_Spark| BRONZE[(Bronze Layer: Delta Table)]
        BRONZE -->|Transformation_Spark| SILVER[(Silver Layer: Delta Table)]
        SILVER -->|Analytics_Spark| GOLD[(Gold Layer: dimensions & facts)]
    end

    AIRFLOW[Lakeflow Jobs]

    AIRFLOW -.-> BRONZE
    AIRFLOW -.-> SILVER
    AIRFLOW -.-> GOLD

```
---

##### Capa Bronce
![Bronze](./images/formula1_bronze_layer.png)

---

##### Capa Silver
![Silver](./images/formula1_silver_layer.png)

---

##### Capa Bronce
![Gold](./images/formula1_gold_layer.png)

---

### 2. Diagrama Carga Incremental

La carga incremental se realiza en base a una tabla de control (*batch_control*) y 4 procesos secuenciales:

- **1. Identify Next Batch**: Analiza si existen nuevos datos en la carpeta de *Landing*.
- **2. Create New Batch**: En caso afirmativo, crea un nuevo registro en la tabla de control.
- **3. Process Batch**: Llama al pipeline completo para procesar el nuevo batch.
- **4. Mark Batch Complete**: Marca el registro como completado en la tabla de control.

```mermaid
graph TD

    subgraph PROCESS_FLOW["Pipeline Execution"]
        IDENTIFY["1. Identify Next Batch"]
        CREATE["2. Create New Batch"]
        
        subgraph MEDALLION["3. Process Batch"]
            BRONZE[("Bronze")] --> SILVER[("Silver")] --> GOLD[("Gold")]
        end
        
        MARK["4. Mark Batch Complete"]

        IDENTIFY -->|batch_id| CREATE
        CREATE -->|IF NEW batch_id THEN execute| MEDALLION
        MEDALLION -->|batch_id| MARK
    end

    subgraph LANDING_AREA["Landing Area"]
        LANDING["✈️ Landing"]
        LANDING_PATH["landing/"]
        BATCH_ID_DIR["batch_id (e.g. 2025-01)"]
        
        CIRCUITS["circuits"]
        RACES["races"]
        CONSTRUCTORS["constructors"]
        DRIVERS["drivers"]
        RESULTS["results"]
        SPRINTS["sprints"]

        LANDING --- LANDING_PATH
        LANDING_PATH --- BATCH_ID_DIR
        BATCH_ID_DIR --- CIRCUITS
        BATCH_ID_DIR --- RACES
        BATCH_ID_DIR --- CONSTRUCTORS
        BATCH_ID_DIR --- DRIVERS
        BATCH_ID_DIR --- RESULTS
        BATCH_ID_DIR --- SPRINTS
    end



    BATCH_CONTROL[("batch_control\n------------------\nbatch_id | batch_status\n2025-01  | complete")]

    %% Conexiones entre estructuras
    BATCH_ID_DIR -->|List of batch_ids| IDENTIFY
    CREATE -->|batch_id| BATCH_CONTROL
    MARK -->|batch_id| BATCH_CONTROL
    BATCH_CONTROL -->|List of batch_ids| IDENTIFY
```
---

## Resultados obtenidos

### 1. Almacenamiento

### - Landing (Azure)

![Landing](./images/azure_storage.png)

### - Landing (Databricks)

![Landing](./images/databricks_landing.png)

### - Capa Bronce

![Bronce](./images/databricks_bronze.png)

### - Capa Silver

![Silver](./images/databricks_silver.png)

### - Capa Gold

![Gold](./images/databricks_gold.png)


### 2. Notebooks

![Notebooks](./images/Workspace_notebooks_folder.png)

### 3. Orquestación

### - Esquema

![Schema](./images/databricks_lakeflow_job.png)

### - Ejecución

![Execution](./images/databricks_lakeflow_job_execution.png)


### 4. Dashboards

### - Driver Standings

![Execution](./images/driver_championship_standings_dashboard.png)

### - Dominant Drivers

![Execution](./images/dominant_drivers_all_time_dashboard.png)

---

## Tech Stack

| Área | Tecnologías Utilizadas |
| :--- | :--- |
| **Compute Engine** | Azure Databricks (PySpark, Spark SQL) |
| **Storage & Lakehouse Layer** | Delta Lake (Bronze ➔ Silver ➔ Gold), ADLS Gen2 |
| **Data Governance** | Databricks Unity Catalog |
| **Orchestration** | Databricks Jobs |
| **Data Analytics & BI** | Databricks Dashboards (AI/BI Dashboards) |
| **Version Control** | Git / GitHub |

---

## Justificación del Stack

Este proyecto ha sido construido con un enfoque puramente práctico para consolidar y validar mis competencias en el ecosistema completo de **Azure** y **Databricks**, trabajando las diferentes herramientas disponibles:

- *Azure Databricks*.
- *Notebooks*.
- *Apache Spark*.
- *Delta Lake + ADLS Gen2*.
- *Unity Catalog*.
- *Lakeflow Jobs*.
- *Databricks Dashboards (AI/BI)*.
- *Genie AI*.

---

## Características principales
- **Aplanado de JSONs Complejos**: Descomposición dinámica de estructuras anidadas.
<br>
- **Uso de Cargas Incrementales (MERGE)**: Actualización eficiente de registros modificados o nuevos sin reescribir tablas completas.
<br>
- **Gestión de Secretos**: Uso de Azure Key Vault y Databricks Secret Scopes para almacenamiento seguro de credenciales.
<br>
- **Calidad de Datos**: Validación de esquemas y tipos de datos en la capa Silver previo al cálculo de agregaciones en Gold.

