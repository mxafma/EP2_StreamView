# Diccionario de KPIs y Reglas de Negocio — StreamView (EP2)

**Responsable:** Michelangelo (P2 — DAX y KPIs)  
**Proyecto:** Evaluación Práctica 2 — Dashboard StreamView  
**Versión del modelo:** `modelo_base_v2.pbix`  

---

## 1. Introducción y Propósito

Este documento consolida la definición técnica, fórmulas DAX, filtros aplicados y objetivos SMART para las 21 medidas calculadas en el modelo multidimensional de **StreamView**. El objetivo principal es garantizar la consistencia analítica entre la capa de datos (`calidad_datos.ipynb`), el modelo relacional en Power BI y la capa de visualización ejecutiva.

---

## 2. Matriz del Diccionario de KPIs

| Grupo | Nombre del KPI | Fórmula DAX | Filtro / Regla Aplicada | Objetivo SMART | Página del Dashboard |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Catálogo** | Total Contenidos | `COUNTROWS('catalogo')` | Ninguno | Contabilizar el 100% de títulos registrados en la plataforma al cierre del periodo. | Resumen / General |
| **Catálogo** | Total Películas | `CALCULATE([Total Contenidos], 'catalogo'[type] = "Movie")` | `type = "Movie"` | Evaluar el volumen total de largometrajes disponibles en la plataforma. | Financiera / Catálogo |
| **Catálogo** | Total Series | `CALCULATE([Total Contenidos], 'catalogo'[type] = "TV Show")` | `type = "TV Show"` | Monitorear la oferta de contenido episódico en el catálogo. | Catálogo / Series |
| **Popularidad** | Popularidad Promedio | `AVERAGE('catalogo'[popularity])` | Ninguno | Mantener un índice de popularidad general promedio superior a 15.0 puntos. | Resumen / Popularidad |
| **Valoración** | Calificación Promedio | `CALCULATE(AVERAGE('catalogo'[vote_average]), 'catalogo'[vote_count] > 0)` | `vote_count > 0` | Mantener una valoración media global superior a 6.5/10 excluyendo títulos sin interacción. | Popularidad y Valoración |
| **Valoración** | Total Votos | `SUM('catalogo'[vote_count])` | Ninguno | Medir el volumen total de interacción y retroalimentación de la audiencia. | Popularidad y Valoración |
| **Valoración** | Calificación Ranking | `CALCULATE([Calificacion Promedio], 'catalogo'[vote_count] >= 'Umbral Votos'[Umbral Votos Valor])` | `vote_count >= Umbral` | Filtrar rankings de calidad considerando únicamente títulos con representatividad estadística. | Popularidad y Valoración |
| **Financieros** | Presupuesto Total | `CALCULATE(SUM('catalogo'[budget]), 'catalogo'[budget] >= 10000)` | `budget >= $10.000` | Auditar la inversión acumulada en producciones y adquisiciones comerciales. | Financiera (películas) |
| **Financieros** | Ingresos Totales | `CALCULATE(SUM('catalogo'[revenue]), 'catalogo'[revenue] >= 10000)` | `revenue >= $10.000` | Cuantificar el retorno bruto total del portafolio comercial de películas. | Financiera (películas) |
| **Financieros** | Presupuesto Promedio | `CALCULATE(AVERAGE('catalogo'[budget]), 'catalogo'[budget] >= 10000)` | `budget >= $10.000` | Mantener el costo promedio de inversión por película por debajo de $30.000.000 USD. | Financiera (películas) |
| **Financieros** | ROI Promedio | `DIVIDE([Ingresos Totales], [Presupuesto Total], 0)` | `budget` e `revenue >= $10.000` | Alcanzar un Retorno de Inversión (ROI) superior al 150% ($1.5\times$) en el agregados comercial. | Financiera (películas) |

---

## 3. Justificación de Reglas de Negocio

1. **Filtro Financiero ($\ge \$10.000$ USD):**  
   Se excluyen los registros con valores de presupuesto o ingresos menores a $10.000 USD para descartar datos nulos, indizados erróneamente con cero ($0$) o producciones independientes sin reporte financiero auditado, previniendo distorsiones graves en el cálculo del ROI.

2. **Filtro de Interacción ($\text{vote\_count} > 0$):**  
   Para la valoración promedio se omiten títulos con cero votos, previniendo que promedien con nota $0.0$ y alteren la percepción real de calidad del catálogo.

3. **Parámetro Dinámico de Ranking (Umbral de Votos):**  
   Se implementó un rango numérico de $0$ a $500$ (incremento de $10$, valor predeterminado en $50$) para que el analista pueda ajustar dinámicamente el sesgo de popularidad en los Top 10 de calificación.

---

## 4. Medidas de Apoyo y Dinamismo

* **`Texto Filtro Tipo`**: Retorna dinámicamente el tipo de contenido seleccionado (`Movie`, `TV Show` o ambos) mediante `SELECTEDVALUE`.
* **`Titulo Dinamico Financiero`**: Construye el encabezado de las gráficas combinando el estado de los filtros activos para mejorar el contexto de lectura ejecutiva.

---

## 5. Validación de Consistencia

Todas las medidas fueron validadas contra la fuente primaria procesada (`data/processed/kpis_referencia.csv`) y los scripts de auditoría en Python (`notebooks/calidad_datos.ipynb`), confirmando una coincidencia del 100% en los totales globales antes de la aplicación de segmentadores.