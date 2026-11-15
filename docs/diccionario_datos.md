# Diccionario de datos: modelo estrella

Tablas generadas en la sección 1.4 del notebook (y `predicciones`, en la sección 3.4) y guardadas
en `data/processed/`. Son las que se cargan en Power BI.

Los montos están en **miles de pesos colombianos**. Un campo vacío significa que el dato no existe o
no se pudo calcular (ver las reglas de negocio al final).

## Relaciones

| Desde | Hacia | Cardinalidad |
|---|---|---|
| `fact_indicadores[nit]` | `dim_empresa[nit]` | Muchos a uno |
| `fact_indicadores[anio]` | `dim_tiempo[anio]` | Muchos a uno |
| `fact_indicadores[id_sector]` | `dim_sector[id_sector]` | Muchos a uno |
| `predicciones[nit]` | `dim_empresa[nit]` | Muchos a uno |
| `predicciones[anio]` | `dim_tiempo[anio]` | Muchos a uno |

El NIT conviene cargarlo como **texto** en todas las tablas para que la relación no falle.

## `fact_indicadores`

Una fila por empresa y año (75.829 filas). Llave: `nit` + `anio`.

| Columna | Tipo | Descripción |
|---|---|---|
| `nit` | Texto | NIT de la empresa, 9 dígitos sin dígito de verificación |
| `anio` | Entero | Año del balance (cierre al 31 de diciembre), de 2019 a 2025 |
| `id_sector` | Entero | Macrosector de la empresa, según el año más reciente en el ranking |
| `activo_corriente` | Número | Activos corrientes totales |
| `activo_total` | Número | Total de activos |
| `pasivo_corriente` | Número | Pasivos corrientes totales. Vacío en 65 balances |
| `pasivo_total` | Número | Total de pasivos |
| `patrimonio_total` | Número | Patrimonio total. Puede ser negativo |
| `efectivo` | Número | Efectivo y equivalentes al efectivo |
| `inventarios` | Número | Inventarios corrientes. Vacío si la empresa no los reporta |
| `deudores_comerciales` | Número | Cuentas comerciales por cobrar y otras cuentas por cobrar corrientes |
| `capital_emitido` | Número | Capital emitido |
| `ganancias_acumuladas` | Número | Ganancias acumuladas |
| `liquidez` | Decimal | Activo corriente / pasivo corriente |
| `endeudamiento` | Decimal | Pasivo total / activo total |
| `solvencia` | Decimal | Patrimonio total / activo total. Es igual a 1 − endeudamiento |
| `capital_trabajo` | Número | Activo corriente − pasivo corriente |
| `prueba_acida` | Decimal | (Activo corriente − inventarios) / pasivo corriente |
| `patrimonio_negativo` | 0 / 1 | 1 si el patrimonio total es menor que cero |
| `ingresos_operacionales_miles_cop` | Número | Ingresos operacionales del ranking del mismo año. Vacío en 2019-2020 y si la empresa no estuvo en el ranking ese año |
| `ganancia_perdida_miles_cop` | Número | Ganancia o pérdida del ranking del mismo año. Mismos vacíos que los ingresos |

## `dim_empresa`

Una fila por empresa (12.761 filas). Llave: `nit`. Los datos salen del año más reciente en que la
empresa aparece en el ranking de las 10.000 empresas.

| Columna | Tipo | Descripción |
|---|---|---|
| `nit` | Texto | NIT de la empresa |
| `razon_social` | Texto | Nombre de la empresa |
| `supervisor` | Texto | Superintendencia que la vigila (Supersociedades, Supersalud, etc.) |
| `region` | Texto | Región del domicilio |
| `departamento` | Texto | Departamento del domicilio |
| `ciudad` | Texto | Ciudad del domicilio |

## `dim_tiempo`

Una fila por año (7 filas). Llave: `anio`.

| Columna | Tipo | Descripción |
|---|---|---|
| `anio` | Entero | Año del balance |
| `fecha_corte` | Fecha | 31 de diciembre de ese año |
| `etapa_covid` | Texto | Antes (2019), Durante (2020-2021) o Después (2022-2025) |

## `dim_sector`

Una fila por macrosector (6 filas). Llave: `id_sector`.

| Columna | Tipo | Descripción |
|---|---|---|
| `id_sector` | Entero | Identificador del macrosector |
| `macrosector` | Texto | Agropecuario, Comercio, Construcción, Manufactura, Minero o Servicios |

## `predicciones`

Probabilidad de deterioro en el año siguiente según el modelo final (regresión logística). Una fila
por empresa y año, el mismo grano de `fact_indicadores`.

| Columna | Tipo | Descripción |
|---|---|---|
| `nit` | Texto | NIT de la empresa |
| `anio` | Entero | Año *t* del balance con el que se calcula la probabilidad |
| `conjunto` | Texto | `entrenamiento` (2019-2022), `prueba` (2023-2024) o `sin objetivo` (2025, o sin año siguiente conocido) |
| `probabilidad_riesgo` | Decimal | Probabilidad de deterioro en *t+1* (patrimonio negativo o liquidez menor que 1), de 0 a 1 |
| `deterioro_t1` | 0 / 1 | Lo que pasó en *t+1*. Vacío si no se conoce |

Las probabilidades de 2025 son el pronóstico para 2026. Las de 2019 a 2022 salen de balances con
los que se entrenó el modelo.

## Reglas de negocio

1. Solo se usan balances de cierre (31 de diciembre) de 2019 a 2025.
2. Si el denominador de un indicador es 0 o está vacío, el indicador queda vacío. No se reemplaza
   por 0 ni por infinito.
3. Si una empresa no reporta inventarios, se toman como 0 solo para calcular la prueba ácida.
4. No se elimina ningún balance por la ecuación contable, porque todos cuadraron.
5. Los balances con patrimonio negativo se conservan y se marcan con `patrimonio_negativo = 1`.
