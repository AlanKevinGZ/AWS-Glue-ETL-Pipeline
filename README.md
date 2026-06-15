# AWS Glue ETL Pipeline — Data Masking & Analytics

Pipeline de procesamiento de datos en la nube usando **AWS Glue** con dos ejercicios prácticos: enmascaramiento de datos sensibles y análisis de datos de películas.

---

## Tecnologías utilizadas

- AWS Glue Studio (Visual ETL — modo gráfico)
- AWS Glue Crawlers + Data Catalog
- AWS S3
- SQL (nodos de transformación en Glue)
- Formato Parquet + compresión GZIP

---

## Ejercicio 1 — Enmascaramiento de datos de clientes (`clientes.csv`)

### Descripción

Job de AWS Glue que procesa un archivo CSV con información personal de clientes aplicando reglas de privacidad de datos.

### Transformaciones aplicadas

| Transformación | Detalle |
|---|---|
| Enmascaramiento PII | Campos `SSN`, `email` y `número de tarjeta de crédito` reemplazados con `#####` |
| Enmascaramiento por fecha | Fechas de nacimiento posteriores al año 2000 reemplazadas con `#####` |
| Ordenamiento | Registros ordenados alfabéticamente por el campo `username` |
| Salida | Formato Parquet con compresión GZIP |

### Estructura en S3

```
s3://bucket-ejercicios/
└── clientes/
    ├── clientes.csv               ← archivo fuente
    └── clientes_procesados/       ← salida del job
        └── *.parquet (GZIP)
```

### Flujo del job

```
S3 (clientes.csv)
    └── Leer con Glue DynamicFrame
        └── Detectar columnas PII (SSN, email, tarjeta)
            └── Aplicar máscara "#####"
                └── Enmascarar fechas de nacimiento > año 2000
                    └── Ordenar por username
                        └── Escribir en S3 (Parquet + GZIP)
```

---

## Ejercicio 2 — Análisis de películas (`movies.parquet`)

### Descripción

Pipeline con **AWS Glue Crawler + ETL Job** que ingiere datos de actores/películas y genera dos reportes analíticos guardados en carpetas separadas dentro de S3.

### Fuente de datos

Archivo `movies.parquet` donde cada fila representa la participación de un actor en una película (relación uno-a-muchos: múltiples filas por película).

### Configuración del Catálogo

- **Base de datos Glue:** `movies`
- **Crawler:** poblado automáticamente desde `s3://bucket-ejercicios/movies/`

### Transformaciones aplicadas

**Reporte 1 — Películas por actor**

| Columna | Descripción |
|---|---|
| `actor` | Nombre del actor |
| `conteo` | Cantidad de películas en las que participó |

Ordenado por `conteo` descendente.

**Reporte 2 — Películas por año**

| Columna | Descripción |
|---|---|
| `año` | Año de producción |
| `siglo` | Siglo al que pertenece el año (ej. `XX`, `XXI`) |
| `conteo` | Cantidad de películas producidas ese año |

Ordenado por `conteo` descendente.

### Estructura en S3

```
s3://bucket-ejercicios/
└── movies/
    ├── movies.parquet                  ← archivo fuente
    ├── peliculas_por_actor/            ← salida reporte 1
    │   └── *.parquet
    └── peliculas_por_anio/             ← salida reporte 2
        └── *.parquet
```

### Flujo del job (Glue Studio — Visual ETL)

El job fue construido con **AWS Glue Studio** en modo visual, con dos ramas paralelas desde una única fuente del Data Catalog:

```
AWS Glue Data Catalog (movies)
│
├── [Rama izquierda — Películas por siglo/año]
│   ├── Transform: SQL Query siglos       ← calcula el siglo a partir del año
│   ├── Transform: Aggregate siglo        ← agrupa y cuenta por siglo
│   ├── Transform: Change Schema          ← renombra/ajusta columnas de salida
│   ├── Transform: SQL Query              ← ordena por conteo DESC
│   └── Data target: Amazon S3            ← escribe reporte peliculas_por_anio/
│
└── [Rama derecha — Películas por actor]
    ├── Transform: Aggregate              ← agrupa y cuenta películas por actor
    ├── Transform: Change Schema          ← renombra/ajusta columnas de salida
    ├── Transform: SQL Query movies c...  ← ordena por conteo DESC
    └── Data target: Amazon S3            ← escribe reporte peliculas_por_actor/
```

> El job utiliza nodos de **SQL Query** para lógica condicional (cálculo de siglo y ordenamiento), **Aggregate** para las agrupaciones y **Change Schema** para estandarizar los esquemas de salida antes de escribir en S3.

---

## Conceptos demostrados

- Diseño de pipelines ETL con **AWS Glue Studio** (modo visual de arrastrar y soltar)
- Uso de **AWS Glue Crawlers** para catalogación automática de datos en S3
- Nodos de transformación: **SQL Query**, **Aggregate**, **Change Schema**
- Lógica de **doble rama paralela** desde una sola fuente del Data Catalog
- Detección y enmascaramiento de **datos PII** (Personally Identifiable Information)
- Escritura en S3 en formato **Parquet con compresión GZIP**
- Columnas derivadas con SQL en Glue (ej. cálculo de siglo a partir del año)

---

## Autor

**Cat** — Frontend Developer en transición a Data Engineering  
Stack objetivo: Python · PySpark · AWS Glue · S3 · Lambda · DynamoDB
