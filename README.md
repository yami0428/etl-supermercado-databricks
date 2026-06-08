# 🛒 ETL Supermercado — Databricks Medallion Architecture

## 📋 Descripción

Pipeline ETL construido en **Azure Databricks** con arquitectura **Medallion** (Bronze → Silver → Golden) usando **PySpark** y datos de un supermercado online (Instacart Dataset).

El proyecto implementa un flujo completo de ingesta, transformación y carga de datos, con visualización en **Power BI** y despliegue automatizado mediante **GitHub Actions (CI/CD)**.

---

## 🏗️ Arquitectura

```
ADLS Gen2 - raw/ (Managed Identity)
       │
       ├── orders.csv
       ├── order_products__prior.csv
       ├── products.csv
       ├── aisles.csv
       └── departments.csv
              │
     ┌────────┴────────┐
     ▼                 ▼
  [BRONZE]         [BRONZE]
  ingest_orders    ingest_productos
     └────────┬────────┘
              ▼
         [SILVER]
         transform.py
         (joins + UDFs + limpieza)
              ▼
          [GOLDEN]
          load.py
          (agregaciones y KPIs)
              ▼
         Power BI Dashboard
```

---

## 🗂️ Estructura del Repositorio

```
├── prepamb/           → SQL: crea catálogo, schemas y tablas (Unity Catalog)
├── proceso/           → ETL en PySpark: ingest, transform, load, orquestador
├── security/          → SQL: GRANTS de acceso por roles
├── reversion/         → SQL: DROP y REVOKE para revertir cambios
├── datasets/          → CSVs fuente (Instacart Supermarket Dataset)
├── dashboard/         → Dashboard Power BI (.pbix) y capturas
├── evidencias/        → Capturas de ejecución y servicios Azure
└── .github/workflows/ → CI/CD con GitHub Actions
```

---

## 📊 Datasets Utilizados

| Archivo | Descripción | Registros |
|---------|-------------|-----------|
| `orders.csv` | Pedidos de usuarios (orden, usuario, día, hora) | 131,209 |
| `order_products__prior.csv` | Productos por pedido | 3,214,874 |
| `products.csv` | Catálogo de productos | 49,688 |
| `aisles.csv` | Pasillos del supermercado | 134 |
| `departments.csv` | Departamentos | 21 |

**Fuente:** [Kaggle — Instacart Market Basket Analysis](https://www.kaggle.com/datasets/amunsentom/supermarket-superstore-dataset-bundle)

---

## 🥉🥈🥇 Capas Medallion

### Bronze — Datos Crudos
- Ingesta directa desde ADLS Gen2 (Managed Identity)
- Sin transformaciones
- Tablas: `orders`, `order_products`, `products`, `aisles`, `departments`

### Silver — Datos Transformados
- Join de las 5 tablas Bronze
- UDFs aplicados: `franja_horaria()`, `nombre_dia()`
- Campo calculado: `es_recompra`
- Tabla: `ventas_detalle`

### Golden — KPIs y Agregaciones
- Tabla `ventas_por_departamento` — productos pedidos y % recompra por departamento
- Tabla `top_productos` — Top 20 productos más populares (Window function + dense_rank)
- Tabla `frecuencia_usuarios` — Segmentación de clientes por frecuencia de compra

---

## ☁️ Servicios Azure

| Servicio | Nombre | Descripción |
|---------|--------|-------------|
| Azure Databricks | `databricks-etl-supermercado-prod` | Workspace de producción |
| Azure Databricks | `databricks-etl-supermercado-dev` | Workspace de desarrollo |
| ADLS Gen2 | `etlsupermercado` | Storage Account con capa raw |
| Access Connector | `ac-etl-supermercado` | Managed Identity para conexión segura |

---

## 🔄 CI/CD — GitHub Actions

El despliegue sigue el flujo **dev → prod**:

```
Rama dev  →  Pull Request  →  Merge a main  →  GitHub Actions
                                                     ↓
                                            Validar sintaxis .py
                                                     ↓
                                            Verificar estructura
                                                     ↓
                                            ✅ Pipeline validado
```




---

## 📈 Tablas Golden (para Dashboard)

| Tabla | Descripción |
|-------|-------------|
| `ventas_por_departamento` | Productos pedidos y % recompra por departamento |
| `top_productos` | Top 20 productos más populares con ranking |
| `frecuencia_usuarios` | Clasificación: Fiel / Frecuente / Ocasional / Nuevo |

---

## 🔐 Seguridad

- Conexión a raw layer mediante **Managed Identity** (sin claves expuestas)
- GRANTs de acceso por roles sobre tablas Golden
- Reversion disponible: `DROP CASCADE` y `REVOKE` de permisos

---

## 👤 Autor

**Francia Yamile** — `fyamile.0428@gmail.com`  
Curso: Data Engineering con Azure Databricks  
Fecha: Junio 2026
