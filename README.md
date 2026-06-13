# Librería Trinidad — Analytics Layer

Capa de transformación de datos construida con dbt Core sobre PostgreSQL, parte del proyecto **Librería Trinidad** — migración completa de una tienda de libros de PHP legacy a stack moderno.

## Ecosistema

| Repositorio                                                             | Descripción                                          |
| ----------------------------------------------------------------------- | ---------------------------------------------------- |
| [libreria-trinidad-api](https://github.com/JuanCosco/libreria-trinidad-api) | REST API con Express + Prisma + PostgreSQL           |
| [libreria-trinidad-web](https://github.com/JuanCosco/libreria-trinidad-web) | Frontend Next.js 16 + Tailwind + shadcn              |
| **libreria-trinidad-analytics** ← estás aquí                                | Capa analytics con dbt sobre la misma DB del backend |

Este repo consume directamente el schema `public` generado por el API y produce modelos analíticos en el schema `analytics`, sin duplicar infraestructura.

## Stack

- **Herramienta:** dbt Core 1.11.11
- **Adaptador:** dbt-postgres 1.10.0
- **Base de datos:** PostgreSQL (tienda_online)
- **Schema de entrada:** `public` (tablas del backend)
- **Schema de salida:** `analytics`

## Arquitectura

public (backend) analytics (dbt output)
```text
├── orders → ├── stg_orders
├── order_items → ├── stg_order_items
├── books → ├── stg_books
├── categories → └── marts/
└── users ├── ventas_por_categoria
├── libros_mas_vendidos
└── resumen_ordenes
```
## Modelos

### Staging

Vistas que limpian y estandarizan las tablas raw del backend.

| Modelo            | Fuente             | Descripción                                        |
| ----------------- | ------------------ | -------------------------------------------------- |
| `stg_orders`      | public.orders      | Órdenes con columnas renombradas y tipos casteados |
| `stg_order_items` | public.order_items | Items con subtotal calculado                       |
| `stg_books`       | public.books       | Libros estandarizados                              |

### Marts

Tablas con métricas de negocio listas para consumir.

| Modelo                 | Descripción                                        |
| ---------------------- | -------------------------------------------------- |
| `ventas_por_categoria` | Revenue, órdenes y unidades vendidas por categoría |
| `libros_mas_vendidos`  | Ranking de libros por cantidad vendida e ingresos  |
| `resumen_ordenes`      | Órdenes agrupadas por estado con métricas de valor |

## Calidad de datos

12 tests cubriendo unicidad de IDs, valores no nulos en columnas críticas y valores aceptados para el campo `status`. Todos en verde.

## Resultados sobre datos reales

El pipeline corre sobre la base de datos del backend en producción local. Muestra de resultados actuales:

| Métrica           | Valor                    |
| ----------------- | ------------------------ |
| Categoría top     | Programación — S/ 994.78 |
| Libro más vendido | Python — 12 unidades     |
| Órdenes pagadas   | 5                        |
| Revenue total     | S/ 1,061.78              |

## Instalación

```bash
pip install dbt-core==1.11.11 dbt-postgres==1.10.0

dbt debug        # verificar conexión
dbt run          # correr modelos
dbt test         # correr tests de calidad
dbt docs generate && dbt docs serve  # documentación interactiva
```

## Configuración profiles.yml

```yaml
tienda_analytics:
  target: dev
  outputs:
    dev:
      type: postgres
      host: localhost
      port: 5432
      user: tu_usuario
      pass: tu_password
      dbname: tienda_online2
      schema: analytics
      threads: 1
```
