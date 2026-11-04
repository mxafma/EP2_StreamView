# Decisiones de limpieza y preparación de datos

Proyecto StreamView Analytics (EP2, ADY1104). Cada caso detectado en los dos CSV, la evidencia, la decisión tomada y dónde se aplica. El detalle con el antes y el después de cada paso está en `notebooks/calidad_datos.ipynb`; el código de Power Query está en `docs/consultas_power_query.md`.

**Dónde se aplica:** *Notebook* = `calidad_datos.ipynb` · *PQ* = consultas de Power Query · *DAX* = medidas del modelo · *Informe* = se declara como limitación o se comenta en la narrativa.

## Tabla de casos y decisiones

| # | Caso | Evidencia | Decisión | Dónde se aplica |
|---|---|---|---|---|
| 1 | `rating` no es la clasificación etaria | Idéntico a `vote_average` en el 100% de las filas de ambos archivos | Excluir la columna. Los KPIs de clasificación etaria se declaran no calculables. | Notebook, PQ, Informe |
| 2 | `duration` inútil | Vacía en películas; dice "1 Seasons" en las 16.000 series | Excluir la columna | Notebook, PQ |
| 3 | `show_id` repetido entre archivos | 397 casos | Clave compuesta `tipo-show_id` | Notebook, PQ |
| 4 | `show_id` repetido dentro de series | 9 títulos con mismo nombre y año, distinta popularidad | Conservar la fila de mayor popularidad (quedan 31.991 filas) | Notebook, PQ |
| 5 | Mismo título y año con distinto `show_id` | 37 películas y 45 series | Mantener: son registros distintos | Notebook |
| 6 | Nulos en `director`, `cast`, `country` y `genres` | Director: 69% de las series y 1% de las películas. País: 11% y 3%. Género: 6% y 1%. | No borrar filas. Rellenar con "Sin dato"; las tablas puente lo excluyen. | Notebook, PQ |
| 7 | Género `Unknown` | 99 series (2 solo con ese género) | Tratarlo como "Sin dato": se excluye de `puente_genero`. Géneros distintos: 27. | Notebook, PQ |
| 8 | Listas de géneros distintas entre películas y series | Action / Action & Adventure; Science Fiction y Fantasy / Sci-Fi & Fantasy; War / War & Politics | No unificar. Analizar géneros por tipo. | Informe, filtros del dashboard |
| 9 | Títulos sin votos | 4.560 (5,6% de las películas y 23% de las series) | Bandera `tiene_votos`; los KPIs de valoración filtran `tiene_votos = TRUE` | Notebook, PQ, DAX |
| 10 | Votos y nota inconsistentes | 30 casos (5 y 12 con votos y nota 0; 13 con nota y 0 votos) | Mantener y declarar | Informe |
| 11 | Pocos votos con notas extremas | 1.017 títulos con menos de 10 votos y nota de 9 o más | Rankings con mínimo de votos (propuesta: 50, quedan 15.397 títulos), como parámetro ajustable | DAX |
| 12 | Ceros en presupuesto e ingresos | Solo 3.492 películas tienen ambos de al menos US$10.000 | Bandera `es_financiero` y `roi` solo para ese subconjunto; los KPIs financieros filtran `es_financiero = TRUE` | Notebook, PQ, DAX |
| 13 | Un solo dato financiero presente | 1.307 con presupuesto e ingresos 0; 2.105 con ingresos y presupuesto 0 | Quedan fuera del subconjunto financiero; declarar la cifra | Informe |
| 14 | Valores menores a US$10.000 | 85 presupuestos y 139 ingresos | Probables errores de carga: quedan fuera del subconjunto | Notebook, PQ |
| 15 | ROI extremo | 24 con más de 50 veces; 77 con menos de 0,01 | Mantener. Mostrar mediana junto al promedio y avisar que el top ROI lo dominan películas de bajo presupuesto. | Informe, visuales |
| 16 | Popularidad con cola larga | ~10% de los títulos son atípicos (IQR); máximo de series: 6.422 | Mantener. Usar mediana o escala logarítmica en distribuciones. | Informe, visuales |
| 17 | Programas diarios en el Top de popularidad | *The Late Show* y *The Tonight Show* encabezan el ranking | Mantener y comentarlo en la narrativa | Informe |
| 18 | Conteo por año plano | 1.000 títulos por año y tipo; `date_added` coincide con el año de estreno | No medir crecimiento por cantidad; mostrar evolución de popularidad, valoración y géneros | Informe, visuales |
| 19 | Un título con varios géneros o países | 67% tiene más de un género; 18% más de un país | Contar en cada uno y avisarlo en el tooltip; los totales por género o país no suman el total de títulos | Informe, visuales |
| 20 | Países con nombre antiguo o con error | `Macedonia`, `Netherlands Antilles`, `Guadaloupe` | **No se modifica.** Revisar cómo los ubica el mapa de Power BI. | Pruebas del dashboard |
| 21 | Noruego con dos códigos | `no` (155 títulos) y `nb` (2 títulos) | **No se modifica.** Se mantienen los 83 idiomas. | — |
| 22 | `cn` y `zh` | 303 y 2.834 títulos | Mantener separados (cantonés y chino) | Notebook, PQ |
| 23 | Idioma `xx` | 9 títulos | Nombrar como "Sin idioma" | Notebook, PQ |
| 24 | Títulos con escritura no latina | 760 | Mantener; comprobar que la fuente del dashboard los muestre | Pruebas del dashboard |
| 25 | Títulos sin país ni género | 612 | Mantener como "Sin dato"; no aparecen en esos análisis | Informe |
| 26 | Espacios sobrantes en texto | 3 celdas en `cast` | Recortar espacios | Notebook, PQ |
| 27 | Fechas y rangos | Sin fechas inválidas, notas fuera de 0–10 ni popularidad menor o igual a 0 | Sin cambios; convertir `date_added` a tipo fecha | Notebook, PQ |
| 28 | Columna `description` | 3.338 nulos y no se usa en el análisis | Excluir de las tablas procesadas | Notebook, PQ |

## Cifras de control

Para validar que el modelo de Power BI coincide con el notebook:

| Elemento | Valor |
|---|---|
| Filas de entrada (películas + series) | 32.000 |
| Filas de `catalogo` | 31.991 |
| Películas | 16.000 |
| Series | 15.991 |
| Títulos con votos | 27.431 |
| Títulos sin votos | 4.560 |
| Películas con `es_financiero = TRUE` | 3.492 |
| Países distintos | 147 |
| Géneros distintos | 27 |
| Idiomas | 83 |
| Filas de `puente_genero` | 65.789 |
| Filas de `puente_pais` | 37.628 |
| Filas de `puente_director` | 24.763 |
| Filas de `puente_actor` | 140.367 |

## Limitaciones de la fuente (no son errores de limpieza)

- No existe clasificación etaria ni duración utilizable.
- Hay exactamente 1.000 títulos por año y tipo, por lo que no se puede medir crecimiento por cantidad.
- Los géneros de películas y series no son comparables entre sí.
- Los datos financieros existen solo para 3.492 películas.
- `popularity` es un índice relativo de TMDB, no reproducciones.
