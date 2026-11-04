# Consultas de Power Query

Cada bloque va en una consulta en blanco (Inicio → Editor avanzado). Cambiar la ruta de los archivos por la del computador donde se abra el proyecto. `peliculas`, `series` y `fnPuente` van con "Habilitar carga" desactivado.

## 1. peliculas
```m
let
  Origen = Csv.Document(File.Contents("C:\Users\PC\Desktop\Tar\EP2_StreamView\Data\raw\netflix_movies_detailed_up_to_2025.csv"),
           [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
  Encabezados = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
  Tipos = Table.TransformColumnTypes(Encabezados,
    {{"show_id", Int64.Type}, {"type", type text}, {"title", type text},
     {"director", type text}, {"cast", type text}, {"country", type text},
     {"date_added", type date}, {"release_year", Int64.Type},
     {"genres", type text}, {"language", type text}, {"description", type text},
     {"popularity", type number}, {"vote_count", Int64.Type},
     {"vote_average", type number}, {"budget", type number}, {"revenue", type number}},
    "en-US"),
  SinInutiles = Table.RemoveColumns(Tipos, {"duration", "rating"})
in
  SinInutiles
```

## 2. series
```m
let
  Origen = Csv.Document(File.Contents("C:\Users\PC\Desktop\Tar\EP2_StreamView\Data\raw\netflix_tv_shows_detailed_up_to_2025.csv"),
           [Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
  Encabezados = Table.PromoteHeaders(Origen, [PromoteAllScalars=true]),
  Tipos = Table.TransformColumnTypes(Encabezados,
    {{"show_id", Int64.Type}, {"type", type text}, {"title", type text},
     {"director", type text}, {"cast", type text}, {"country", type text},
     {"date_added", type date}, {"release_year", Int64.Type},
     {"genres", type text}, {"language", type text}, {"description", type text},
     {"popularity", type number}, {"vote_count", Int64.Type},
     {"vote_average", type number}},
    "en-US"),
  SinInutiles = Table.RemoveColumns(Tipos, {"duration", "rating"})
in
  SinInutiles
```

## 3. catalogo
```m
let
  Anexo = Table.Combine({peliculas, series}),
  Clave = Table.AddColumn(Anexo, "clave", each [type] & "-" & Text.From([show_id]), type text),
  Ordenado = Table.Buffer(Table.Sort(Clave, {{"popularity", Order.Descending}})),
  SinDuplicados = Table.Distinct(Ordenado, {"clave"}),
  TieneVotos = Table.AddColumn(SinDuplicados, "tiene_votos", each [vote_count] > 0, type logical),
  EsFinanciero = Table.AddColumn(TieneVotos, "es_financiero",
    each [type] = "Movie" and [budget] <> null and [revenue] <> null
         and [budget] >= 10000 and [revenue] >= 10000, type logical)
in
  EsFinanciero
```

## 4. fnPuente (función)
```m
(columna as text, nombre as text) as table =>
let
  Base = Table.SelectColumns(catalogo, {"clave", columna}),
  SinVacios = Table.SelectRows(Base, each Record.Field(_, columna) <> null
                                       and Record.Field(_, columna) <> ""),
  Lista = Table.TransformColumns(SinVacios, {{columna, each Text.Split(_, ", "), type list}}),
  Filas = Table.ExpandListColumn(Lista, columna),
  Limpio = Table.TransformColumns(Filas, {{columna, each Text.Trim(_), type text}}),
  Unico = Table.Distinct(Limpio),
  Renombrado = Table.RenameColumns(Unico, {{columna, nombre}})
in
  Renombrado
```

## 5–8. Tablas puente
```m
// puente_genero
let Origen = fnPuente("genres", "genero") in Origen
// puente_pais
let Origen = fnPuente("country", "pais") in Origen
// puente_director
let Origen = fnPuente("director", "director") in Origen
// puente_actor
let Origen = fnPuente("cast", "actor") in Origen
```

## 9. dim_idioma
```m
let
  Origen = Table.FromRows(
    {{"af","Afrikáans"}, {"am","Amárico"}, {"ar","Árabe"}, {"as","Asamés"},
     {"az","Azerí"}, {"bg","Búlgaro"}, {"bn","Bengalí"}, {"bs","Bosnio"},
     {"ca","Catalán"}, {"cn","Cantonés"}, {"cs","Checo"}, {"cy","Galés"},
     {"da","Danés"}, {"de","Alemán"}, {"dz","Dzongkha"}, {"el","Griego"},
     {"en","Inglés"}, {"es","Español"}, {"et","Estonio"}, {"eu","Vasco"},
     {"fa","Persa"}, {"fi","Finés"}, {"fr","Francés"}, {"ga","Irlandés"},
     {"gl","Gallego"}, {"he","Hebreo"}, {"hi","Hindi"}, {"hr","Croata"},
     {"ht","Criollo haitiano"}, {"hu","Húngaro"}, {"hy","Armenio"}, {"id","Indonesio"},
     {"is","Islandés"}, {"it","Italiano"}, {"ja","Japonés"}, {"ka","Georgiano"},
     {"kk","Kazajo"}, {"kl","Groenlandés"}, {"km","Jemer"}, {"kn","Canarés"},
     {"ko","Coreano"}, {"ku","Kurdo"}, {"ky","Kirguís"}, {"la","Latín"},
     {"lb","Luxemburgués"}, {"lt","Lituano"}, {"lv","Letón"}, {"mi","Maorí"},
     {"mk","Macedonio"}, {"ml","Malayalam"}, {"mn","Mongol"}, {"mr","Maratí"},
     {"ms","Malayo"}, {"mt","Maltés"}, {"nb","Noruego bokmål"}, {"ne","Nepalí"},
     {"nl","Neerlandés"}, {"no","Noruego"}, {"or","Odia"}, {"pa","Panyabí"},
     {"pl","Polaco"}, {"ps","Pastún"}, {"pt","Portugués"}, {"ro","Rumano"},
     {"ru","Ruso"}, {"si","Cingalés"}, {"sk","Eslovaco"}, {"sl","Esloveno"},
     {"sr","Serbio"}, {"sv","Sueco"}, {"ta","Tamil"}, {"te","Telugu"},
     {"th","Tailandés"}, {"tl","Tagalo"}, {"tr","Turco"}, {"uk","Ucraniano"},
     {"ur","Urdu"}, {"vi","Vietnamita"}, {"xx","Sin idioma"}, {"yo","Yoruba"},
     {"za","Zhuang"}, {"zh","Chino"}, {"zu","Zulú"}},
    {"codigo_idioma", "idioma"})
in
  Origen
```
