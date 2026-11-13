# Semáforo de salud financiera empresarial

Proyecto integrador — Inteligencia de Negocios · Análisis de Datos · Aprendizaje Computacional (ITM).

**Pregunta de negocio:** con el balance de una empresa en el año *t*, ¿podemos anticipar si estará
en deterioro financiero en el año *t+1*?

**Equipo:** Julián, Santiago, Jacobo y Oscar.

## Fuentes de datos (datos.gov.co)

| Base | Publica | Uso |
|---|---|---|
| [Estados Financieros NIIF – Estado de Situación Financiera](https://www.datos.gov.co/d/pfdp-zks5) | Supersociedades | Balances por empresa y año |
| [10.000 Empresas más Grandes del País](https://www.datos.gov.co/d/6cat-2gcs) | Supersociedades | Sector, región, ciudad, ingresos y ganancia |

## Estructura

```
data/raw/          descargas crudas (no se suben al repositorio)
data/processed/    datos limpios y tablas del modelo estrella
notebooks/         proyecto_integrador.ipynb (una sección por etapa)
dashboard/         semaforo_financiero.pbix
docs/              diccionario de datos y capturas del dashboard
```

## Cómo ejecutar

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate   |   Linux/Mac: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/proyecto_integrador.ipynb
```

Opcional: crear un token gratuito en datos.gov.co y guardarlo en la variable de entorno
`DATOS_GOV_TOKEN` para que la API no limite las descargas.

La sección 1 del notebook descarga los datos a `data/raw/`. Las secciones siguientes trabajan
con los archivos de `data/processed/`.

## Plan de trabajo

| Etapa | Julián | Santiago | Jacobo | Oscar |
|---|---|---|---|---|
| 1. ETL | C1 Extracción por API | C2 Limpieza y pivoteo | C3 Integración y validación | C4 Indicadores y modelo estrella |
| 2. EDA | C5 Descriptiva | C6 Tendencias | C7 Correlaciones y anomalías | C8 Pruebas de hipótesis |
| 3. ML | C9 Variable objetivo | C10 Escalado, codificación y PCA | C11 Entrenamiento | C12 Evaluación |
| 4. BI y cierre | C13 Modelo de datos BI | C14 Vistas directiva y sectorial | C15 Semáforo por empresa | C16 Conclusiones y ética |
