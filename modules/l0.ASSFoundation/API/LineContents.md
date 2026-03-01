# LineContents (l0.ASSFoundation) — v0.5.0

`LineContents` representa una línea ASS parseada en **secciones** (`Text`, `Tag`, `Drawing`, `Comment`) y ofrece API para:

- reconstruir el string final,
- insertar/remover/reemplazar tags,
- calcular tags efectivos,
- dividir (split) por tags/intervalos/índices,
- calcular métricas/bounds (SubInspector),
- utilidades (reverse/trim/strip).

---

## Constructor y refs

### `LineContents.new(line, sections?, copyAndCheckSections=true) -> LineContents`

**Hace:** construye un LineContents desde una `Line`. Si `sections` no se dan, parsea usando el parser de ASSFoundation.  
**Notas:** valida que `line` sea `Line`. Puede mantener referencias a `sub/styles/scriptInfo` si la línea viene de un `LineCollection`.

### `LineContents.updateRefs(prevCnt) -> boolean`

**Hace:** actualiza `prevSection`, `parent`, `index` de cada sección cuando cambia el número de secciones.

---

## Serialización / obtención de secciones

### `LineContents.getString(includeEmptySections=true, currDrawingState?, predicate?, predicateLookAhead=true, a1?, a2?, a3?) -> (string, drawingState)`

**Hace:** reconstruye el texto ASS (con `{...}`) desde las secciones.

- Inserta automáticamente `\p` cuando cambia entre Text y Drawing para mantener “drawing state” consistente.
- `predicate` puede ser función o lista de clases para filtrar secciones.

### `LineContents.get(sectionClasses?, start?, end_?, relative?) -> {Section...}`

**Hace:** devuelve copias (`copy!`) de secciones que matcheen clase(s), usando internamente `callback`.

### `LineContents.callback(callbackFn, sectionClasses?, start=1, end_?, relative?, reverse?) -> number|false`

**Hace:** iterador/modificador general sobre secciones.

- Permite rangos absolutos o “relativos” (por conteo de coincidencias).
- Acepta índices negativos para iteración desde el final (y ajusta `reverse`).

**Contrato del callback:**
- `false` ⇒ elimina la sección
- `true` / `nil` ⇒ deja igual
- cualquier otro valor ⇒ reemplaza la sección por ese valor

### `LineContents.getSectionCount(classes) -> number`

**Hace:** cuenta secciones por clase(s).

---

## Insertar / remover secciones

### `LineContents.insertSections(sections, index?) -> {sections...}`

**Hace:** inserta una o varias secciones en posición `index` (por defecto al final + 1). Valida que sean instancias de `ASS.Section.*`.

### `LineContents.removeSections(start, end_=start) -> {removed...}`

**Hace:** elimina secciones por:

- `nil` ⇒ purga todo
- `number` ⇒ rango
- `table` / obj ⇒ por lista/set de objetos

Limpia `parent/index/prevSection` de las removidas.

---

## Tags: contar / iterar / insertar / remover / reemplazar

### `LineContents.getTagCount() -> number`

**Hace:** cuenta tags totales sumando tags en secciones Tag.

### `LineContents.modTags(tagNames, callback, start=1, end_?, relative?) -> number|false`

**Hace:** recorre tags con nombre(s) `tagNames` a través de secciones Tag y aplica callback.  
**Notas:** maneja rangos relativos y negativos (desde el final).

### `LineContents.getTags(tagNames, start?, end_?, relative?) -> {Tag...}`

**Hace:** recolecta tags encontrados (usando `modTags`).

### `LineContents.replaceTags(tagList, start?, end_?, relative?, insertRemaining=true) -> nil`

**Hace:** “reemplaza si existe, e inserta si falta”.

- Acepta `Tag`, `TagList`, `Section.Tag`, tabla compatible.
- Reemplaza tags encontrados y prepara “restantes” para inserción.
- Trata tags globales: los inserta siempre en la primera sección Tag (creándola si hace falta).
- Si `insertRemaining` es true y `start==1` (no relativo) y no hay sección Tag inicial, la crea.

### `LineContents.removeTags(tags, start=1, end_?, relative?) -> {removed...}`

**Hace:** elimina tags por nombre(s) o especificación, respetando rango/relativo/negativos, y devuelve los removidos.

### `LineContents.insertTags(tags, index=1, sectionPosition?, direct?) -> ???`

**Hace:** inserta tags en una sección Tag objetivo (o la crea/elige según `sectionPosition`).  
**Nota:** tiene validaciones de índice y de “target section type”.

### `LineContents.insertDefaultTags(tagNames, index?, sectionPosition?, direct?) -> ???`

**Hace:** inserta tags “default” (derivados de estilo) para los `tagNames` indicados.

### `LineContents.getEffectiveTags(index=1, includeDefault?, includePrevious?, copyTags=true) -> TagList|...`

**Hace:** calcula tags efectivos “a partir de” una sección (soporta índice negativo), con opción de incluir defaults y/o tags previos.  
**Nota:** valida índice != 0.

---

## Limpieza / strip

### `LineContents.stripTags() -> self`
### `LineContents.stripText() -> self`
### `LineContents.stripComments() -> self`
### `LineContents.stripDrawings() -> self`

**Hace:** elimina secciones completas del tipo indicado.

### `LineContents.cleanTags(level=3, mergeConsecutiveSections=true, defaultToKeep?, tagSortOrder?) -> self|...`

**Hace:** normaliza/ordena/mergea secciones Tag según nivel, opcionalmente fusionando secciones consecutivas.

### `LineContents.trim() -> self`

**Hace:** recorta/limpia extremos (principalmente sobre secciones de texto).

---

## Split (divide una línea en varias)

### `LineContents.splitAtTags(cleanLevel=3, reposition?, writeOrigin?) -> {LineContents...}|...`

**Hace:** splittea cuando encuentra tags (útil para fbf / efectos), con limpieza y opción de reposicionar y escribir origen.

### `LineContents.splitAtIntervals(callbackOrNumber, cleanLevel=3, reposition=true, writeOrigin) -> {LineContents...}|...`

**Hace:** split por intervalos:

- si el primer argumento es número ⇒ intervalos fijos
- si es callback ⇒ debe devolver número (índice/avance) y debe ser creciente

### `LineContents.splitAtIndexes(indices, cleanLevel?, reposition?, writeOrigin?) -> {LineContents...}|...`

**Hace:** split por una lista de índices (o un índice único). Valida tipo.

### `LineContents.repositionSplitLines(splitLines, writeOrigin=true) -> {splitLines...}`

**Hace:** ajusta posición/origen de líneas resultantes del split.

---

## Commit / undo

### `LineContents.commit(line=@line, includeEmptySections=true, text=@getString(includeEmptySections)) -> string`

**Hace:** escribe el texto al `line.text`, guarda `line.undoText`, y regenera `raw` (`line:createRaw!`). Devuelve el texto final.

### `LineContents.undoCommit(line=@line) -> boolean`

**Hace:** revierte a `undoText` si existe y regenera raw.

---

## Estilo / posición / métricas

### `LineContents.getStyleRef(style) -> styleRef`

**Hace:** resuelve un style por nombre o styleRef; error si no existe.

### `LineContents.getPosition(style?, align?, forceDefault?) -> ASS.Point`

**Hace:** calcula posición considerando tags efectivos y/o estilo; valida tipo de `align`.

### `LineContents.getDefaultTags(style?, copyTags=true, useOvrAlign=true) -> TagList`

**Hace:** construye TagList default desde estilo.

### `LineContents.getTextExtents(coerce?) -> (w,h,descent,extlead?)|...`

**Hace:** mide texto.

### `LineContents.getLineBounds(noCommit?, keepRawText?) -> Bounds|...`

**Hace:** calcula bounds usando SubInspector.

### `LineContents.getTextMetrics(calculateBounds?, coerce?) -> metrics`

**Hace:** devuelve métricas (y opcionalmente bounds).

### `LineContents.getTextLength() -> number`

**Hace:** suma longitud de secciones Text.

---

## Otros

### `LineContents.isAnimated() -> boolean`

**Hace:** detecta si la línea tiene animación.

### `LineContents.reverse() -> self`

**Hace:** invierte el contenido manteniendo consistencia.