# Parser.LineText (l0.ASSFoundation) — v0.5.0

Este parser se encarga de dividir `line.text` en secciones de contenido:

- `Section.Text`
- `Section.Tag` / `Section.Comment` (cuando detecta `{...}`)
- `Section.Drawing` (si está activo `\pN`)

La idea: construir un arreglo de secciones “listas para usar” por `LineContents`.

---

# Estado de dibujo (`\pN`)

El parser mantiene un estado llamado `drawingState` que es un tag `drawing` (`\p0`, `\p1`, ...).

- Si `drawingState.value == 0` ⇒ el texto se interpreta como `Section.Text`.
- Si `drawingState.value != 0` ⇒ el texto se interpreta como `Section.Drawing{ str=..., scale=drawingState }`.

Esto es clave: la misma “cadena” puede ser texto o comandos de dibujo dependiendo del último `\pN` visto.

---

# API

## `getSections(line) -> {Section...}`

Parsea `line.text` y devuelve un arreglo de secciones.

### Algoritmo (alto nivel)

1) Recorre el texto buscando overrides con el patrón `{.-}`.
2) Si hay texto **antes** del override:
   - lo manda a `insertContentSections(...)` (Text o Drawing según `drawingState`).
3) El contenido dentro del override (`rawTags`) lo convierte con:
   - `ASS.Parser.Sections.getTagOrCommentSection(rawTags)`.

4) Si el resultado es `Section.Tag`:
   - elimina tags `drawing` del TagSection con `removeTags("drawing")`
   - si el TagSection se quedó vacío y solo eran tags `\pN`, descarta ese TagSection
   - actualiza `drawingState` al último `\pN` removido (si hubo), o conserva el anterior.

5) Agrega el TagSection/CommentSection (si quedó alguno).
6) Continúa hasta consumir todo el texto.

---

## `getLineContents(line) -> LineContents`

Atajo:

- construye `LineContents(line, getSections(line), false)`

El `false` final es un flag interno (en v0.5.0 se pasa tal cual desde el parser).

---

# Helper interno (relevante para entender el output)

## `insertContentSections(str, sections, sectCnt, drawingState) -> newSectCnt`

Inserta una sección de contenido en `sections`:

- Si `drawingState.value == 0`:
  - inserta `ASS.Section.Text(str)`
- Si `drawingState.value != 0`:
  - inserta `ASS.Section.Drawing{ str=str, scale=drawingState }`
  - si la `DrawingSection` detecta `junk`, esa “basura” se inserta como sección adicional (y aumenta el contador).

En otras palabras: el parser puede producir `Drawing + junk` como dos secciones separadas.