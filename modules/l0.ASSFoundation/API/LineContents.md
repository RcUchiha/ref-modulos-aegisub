# LineContents (l0.ASSFoundation) — v0.5.0

`LineContents` representa una línea ASS parseada en **secciones** (Text/Tag/Drawing/Comment) y ofrece API para:
- reconstruir el string final,
- insertar/remover/reemplazar tags,
- calcular tags efectivos,
- splittear por tags/intervalos/índices,
- calcular métricas/bounds (SubInspector),
- utilidades (reverse/trim/strip).

---

## Constructor y refs

### `LineContents.new(line, sections?, copyAndCheckSections=true) -> LineContents`
**Hace:** construye un LineContents desde una `Line`. Si `sections` no se dan, parsea usando el parser de ASSFoundation. Mantiene referencias a `sub/styles/scriptInfo` si la línea viene de un `LineCollection`.  
**Notas:** valida que `line` sea `Line`. :contentReference[oaicite:2]{index=2}

### `LineContents.updateRefs(prevCnt) -> boolean`
**Hace:** actualiza `prevSection`, `parent`, `index` de cada sección cuando cambia el número de secciones. :contentReference[oaicite:3]{index=3}

---

## Serialización / obtención de secciones

### `LineContents.getString(includeEmptySections=true, currDrawingState?, predicate?, predicateLookAhead=true, a1?, a2?, a3?) -> (string, drawingState)`
**Hace:** reconstruye el texto ASS (con `{...}`) desde las secciones.
- Inserta automáticamente `\p` (drawing tag) cuando cambia entre Text y Drawing para mantener “drawing state” consistente.
- `predicate` puede ser función o lista de clases para filtrar secciones. :contentReference[oaicite:4]{index=4}

### `LineContents.get(sectionClasses?, start?, end_?, relative?) -> {Section...}`
**Hace:** devuelve copias (`copy!`) de secciones que matcheen clase(s), usando internamente `callback`. :contentReference[oaicite:5]{index=5}

### `LineContents.callback(callbackFn, sectionClasses?, start=1, end_?, relative?, reverse?) -> number|false`
**Hace:** iterador/modificador general sobre secciones.
- Permite rangos absolutos o “relativos” (por conteo de coincidencias).
- Acepta índices negativos para iteración desde el final (y ajusta `reverse`).  
**Contrato del callback:**
- `false` => elimina la sección
- `true`/`nil` => deja igual
- cualquier otro valor => reemplaza la sección por ese valor :contentReference[oaicite:6]{index=6}

### `LineContents.getSectionCount(classes) -> number`
**Hace:** cuenta secciones por clase(s). :contentReference[oaicite:7]{index=7}

---

## Insertar / remover secciones

### `LineContents.insertSections(sections, index?) -> {sections...}`
**Hace:** inserta una o varias secciones en posición `index` (por defecto al final + 1). Valida que sean instancias de `ASS.Section.*`. :contentReference[oaicite:8]{index=8}

### `LineContents.removeSections(start, end_=start) -> {removed...}`
**Hace:** elimina secciones por:
- `nil` => purga todo
- `number` => rango
- `table`/obj => por lista/set de objetos  
Limpia `parent/index/prevSection` de las removidas. :contentReference[oaicite:9]{index=9}

---

## Tags: contar / iterar / insertar / remover / reemplazar

### `LineContents.getTagCount() -> number`
**Hace:** cuenta tags totales sumando tags en secciones Tag. :contentReference[oaicite:10]{index=10}

### `LineContents.modTags(tagNames, callback, start=1, end_?, relative?) -> number|false`
**Hace:** recorre tags con nombre(s) `tagNames` a través de secciones Tag y aplica callback.
**Notas:** maneja rangos relativos y negativos (desde el final). :contentReference[oaicite:11]{index=11}

### `LineContents.getTags(tagNames, start?, end_?, relative?) -> {Tag...}`
**Hace:** recolecta tags encontrados (usando `modTags`). :contentReference[oaicite:12]{index=12}

### `LineContents.replaceTags(tagList, start?, end_?, relative?, insertRemaining=true) -> nil`
**Hace:** “reemplaza si existe, e inserta si falta”:
- Acepta `Tag`, `TagList`, `Section.Tag`, tabla compatible.
- Reemplaza tags encontrados y prepara “restantes” para inserción.
- Trata tags globales: los inserta siempre en la primera sección Tag (creándola si hace falta).
- Si `insertRemaining` es true y `start==1` (no relativo) y no hay sección Tag inicial, la crea. :contentReference[oaicite:13]{index=13}

### `LineContents.removeTags(tags, start=1, end_?, relative?) -> {removed...}`
**Hace:** elimina tags por nombre(s) o especificación, respetando rango/relativo/negativos, y devuelve los removidos. :contentReference[oaicite:14]{index=14}

### `LineContents.insertTags(tags, index=1, sectionPosition?, direct?) -> ???`
**Hace:** inserta tags en una sección Tag objetivo (o la crea/elige según `sectionPosition`).  
**Nota:** este método tiene validaciones de índice y de “target section type”. :contentReference[oaicite:15]{index=15}

### `LineContents.insertDefaultTags(tagNames, index?, sectionPosition?, direct?) -> ???`
**Hace:** inserta tags “default” (derivados de estilo) para los `tagNames` indicados. :contentReference[oaicite:16]{index=16}

### `LineContents.getEffectiveTags(index=1, includeDefault?, includePrevious?, copyTags=true) -> TagList|...`
**Hace:** calcula tags efectivos “a partir de” una sección (soporta índice negativo), con opción de incluir defaults y/o tags previos.  
**Nota:** valida índice != 0. :contentReference[oaicite:17]{index=17}

---

## Limpieza / strip

### `LineContents.stripTags() -> self`
### `LineContents.stripText() -> self`
### `LineContents.stripComments() -> self`
### `LineContents.stripDrawings() -> self`
**Hace:** elimina secciones completas del tipo indicado. :contentReference[oaicite:18]{index=18}

### `LineContents.cleanTags(level=3, mergeConsecutiveSections=true, defaultToKeep?, tagSortOrder?) -> self|...`
**Hace:** normaliza/ordena/mergea secciones Tag según nivel, opcionalmente fusionando secciones consecutivas (con predicate si pasas función). :contentReference[oaicite:19]{index=19}

### `LineContents.trim() -> self`
**Hace:** recorta/limpia extremos (principalmente sobre secciones de texto). :contentReference[oaicite:20]{index=20}

---

## Split (divide una línea en varias)

### `LineContents.splitAtTags(cleanLevel=3, reposition?, writeOrigin?) -> {LineContents...}|...`
**Hace:** splittea cuando encuentra tags (útil para fbf / efectos), con limpieza y opción de reposicionar y escribir origen. :contentReference[oaicite:21]{index=21}

### `LineContents.splitAtIntervals(callbackOrNumber, cleanLevel=3, reposition=true, writeOrigin) -> {LineContents...}|...`
**Hace:** split por intervalos:
- si el primer argumento es número => intervalos fijos
- si es callback => debe devolver número (índice/avance) y debe ser creciente  
Tiene validaciones explícitas para esto. :contentReference[oaicite:22]{index=22}

### `LineContents.splitAtIndexes(indices, cleanLevel?, reposition?, writeOrigin?) -> {LineContents...}|...`
**Hace:** split por una lista de índices (o un índice único). Valida tipo. :contentReference[oaicite:23]{index=23}

### `LineContents.repositionSplitLines(splitLines, writeOrigin=true) -> {splitLines...}`
**Hace:** ajusta posición/origen de líneas resultantes del split. :contentReference[oaicite:24]{index=24}

---

## Commit / undo

### `LineContents.commit(line=@line, includeEmptySections=true, text=@getString(includeEmptySections)) -> string`
**Hace:** escribe el texto al `line.text`, guarda `line.undoText`, y regenera “raw” (`line:createRaw!`). Devuelve el texto final. :contentReference[oaicite:25]{index=25}

### `LineContents.undoCommit(line=@line) -> boolean`
**Hace:** revierte a `undoText` si existe y regenera raw. :contentReference[oaicite:26]{index=26}

---

## Estilo / posición / métricas

### `LineContents.getStyleRef(style) -> styleRef`
**Hace:** resuelve un style por nombre o styleRef; error si no existe. :contentReference[oaicite:27]{index=27}

### `LineContents.getPosition(style?, align?, forceDefault?) -> ASS.Point`
**Hace:** calcula posición considerando tags efectivos y/o estilo; valida tipo de `align`. :contentReference[oaicite:28]{index=28}

### `LineContents.getDefaultTags(style?, copyTags=true, useOvrAlign=true) -> TagList`
**Hace:** construye TagList default desde estilo, con opciones de copia y alineación. :contentReference[oaicite:29]{index=29}

### `LineContents.getTextExtents(coerce?) -> (w,h,descent,extlead?)|...`
**Hace:** mide texto (probablemente usando utilidades/font metrics); `coerce` fuerza defaults. :contentReference[oaicite:30]{index=30}

### `LineContents.getLineBounds(noCommit?, keepRawText?) -> Bounds|...`
**Hace:** calcula bounds usando SubInspector; maneja errores/mensajes y cache de “sub”. :contentReference[oaicite:31]{index=31}

### `LineContents.getTextMetrics(calculateBounds?, coerce?) -> metrics`
**Hace:** devuelve métricas (y opcionalmente bounds). :contentReference[oaicite:32]{index=32}

### `LineContents.getTextLength() -> number`
**Hace:** suma `len` de secciones Text (similar a la propiedad `textLength`). :contentReference[oaicite:33]{index=33}

---

## Otros

### `LineContents.isAnimated() -> boolean`
**Hace:** detecta si la línea tiene animación (típicamente por transforms/variación). :contentReference[oaicite:34]{index=34}

### `LineContents.reverse() -> self`
**Hace:** invierte el contenido (reordena secciones) manteniendo consistencia de secciones Tag/texto. :contentReference[oaicite:35]{index=35}
