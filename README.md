# StreamView Analytics – EP2 (ADY1104 Visualización de Datos)

Dashboard interactivo en Power BI para StreamView Analytics. Corresponde a la Evaluación Parcial 2 (Diseño de Dashboards) del caso semestral.

## Estructura de la carpeta

```
EP2_StreamView/
├── data/raw/        CSV originales sin modificar
├── data/processed/  tablas procesadas por el notebook (catalogo, puentes, dim_idioma, kpis_referencia)
├── notebooks/       calidad_datos.ipynb (calidad de datos y antes/después de cada transformación)
├── dashboard/       StreamView_EP2.pbix + exportación PDF
├── images/          gráficos estáticos y capturas
├── docs/            informe, presentación, diccionario de KPIs, decisiones_limpieza.md, consultas de Power Query
└── README.md
```

## Cómo abrir el proyecto

1. Instalar Power BI Desktop (Windows).
2. Abrir `dashboard/modelo_base.pbix` (luego `StreamView_EP2.pbix`).
3. Si Power BI no encuentra los datos: Inicio → Transformar datos → en las consultas `peliculas` y `series`, editar la ruta del archivo (Editor avanzado) para que apunte a `data/raw/` en tu computador.
4. Inicio → Cerrar y aplicar.

El código de todas las consultas está en `docs/consultas_power_query.md`. La tabla de casos y decisiones de limpieza está en `docs/decisiones_limpieza.md`.

## Modelo de datos

| Tabla | Descripción |
|---|---|
| `catalogo` | Películas y series anexadas, con clave compuesta `tipo-show_id` |
| `puente_genero`, `puente_pais`, `puente_director`, `puente_actor` | Una fila por título y valor (uno a varios desde `catalogo[clave]`) |
| `dim_idioma` | Código ISO → nombre del idioma (83 idiomas) |

Relaciones: `catalogo[clave]` → cada tabla puente (1:*), y `dim_idioma[codigo_idioma]` → `catalogo[language]` (1:*), todas con filtro en una dirección.

## Transformaciones realizadas (Power Query)

1. Importación de los dos CSV con `QuoteStyle.Csv` y cultura `en-US` para los decimales.
2. Se eliminan `duration` y `rating` (ver limitaciones).
3. Se anexan películas y series en `catalogo` y se crea la clave compuesta `tipo-show_id`.
4. Se eliminan 9 `show_id` duplicados dentro de series, conservando la fila de mayor `popularity`.
5. Se recortan espacios y los nulos de `director`, `cast`, `country` y `genres` pasan a "Sin dato".
6. Se agregan `tiene_votos` (`vote_count > 0`), `es_financiero` (película con `budget` y `revenue` ≥ US$10.000) y `roi` (solo para las financieras). Se quita `description`.
7. `genres`, `country`, `director` y `cast` se separan por ", " en tablas puente, excluyendo "Sin dato" y el género `Unknown`.
8. `language` (código ISO) se traduce a nombre con `dim_idioma`.

Cifras de control: 32.000 filas de entrada, 31.991 en `catalogo`, 3.492 películas financieras, 27 géneros, 147 países y 83 idiomas.

## Limitaciones de los datos

- `rating` es idéntico a `vote_average` en el 100% de las filas, por lo que no existe clasificación etaria y esos KPIs no se calculan.
- `duration` está vacía en películas y dice "1 Seasons" en todas las series.
- Hay exactamente 1.000 títulos por año y tipo (2010–2025); el conteo por año es plano.
- 5,6% de las películas y 23% de las series tienen `vote_count = 0`; se excluyen de promedios de valoración.
- Solo 3.492 películas tienen presupuesto e ingresos de al menos US$10.000.
- Un título con varios géneros o países cuenta en cada uno.
- Películas y series usan listas de géneros distintas: los géneros se analizan por tipo.
- Los títulos sin país o género ("Sin dato") no aparecen en esos análisis.
- `popularity` es un índice relativo de TMDB, no reproducciones.

## Equipo

| Rol | Integrante |
|---|---|
| P1 · Datos, modelo y repositorio | Matías |
| P2 · DAX y KPIs | Michelangelo |
| P3 · Interacción y navegación | Seba |
| P4 · Diseño, UX, exportación y publicación | Hernán |
