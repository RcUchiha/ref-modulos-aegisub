# Parser.Sections (l0.ASSFoundation) — v0.5.0

Este parser se encarga de interpretar el contenido **dentro de `{...}`** y decidir si eso representa:

- una `Section.Tag` (tags parseables), o
- una `Section.Comment` (basura/comentarios dentro de llaves)

En otras palabras: es el “filtro” que traduce el raw override string a secciones estructuradas.

---

## Cómo detecta tags

Usa un patrón regex (interno) para iterar por posibles tags:

- Detecta secuencias tipo `\tag...`
- Soporta parámetros con paréntesis tipo `\t(...)`
- Tolera “restos” después de un match (los convierte en `unknown` o `junk`)

---

# API

## `getTagOrCommentSection(rawTags) -> Section.Tag | Section.Comment`

Recibe el string **sin llaves**, o sea lo que va dentro de `{ ... }`.

### Flujo

1) Llama a `parseTags(rawTags)` para obtener una lista de tags.  
2) Si el resultado **no tiene tags** pero `rawTags` **no está vacío**, devuelve `Section.Comment(rawTags)`.  
3) Si sí hay tags, devuelve `Section.Tag(tags)`.

Esto permite distinguir entre:

- `{\\bord2\\fs40}` → `Section.Tag`
- `{esto no son tags}` → `Section.Comment`

---

## `parseTags(rawTags) -> {Tag...}`

Parsea el contenido raw (sin llaves) y devuelve un arreglo de objetos `Tag`.

### Comportamiento

- Si `rawTags` está vacío, retorna `{}`.
- Recorre coincidencias con el patrón de tags.
- Cada match lo procesa con `ASS.getTagFromString(match)` y agrega el tag.
- Si dentro del match quedó “texto sobrante” (ej. comentario pegado), lo convierte a:
  - `ASS.createTag("unknown", ...)` si empieza con `\`
  - `ASS.createTag("junk", ...)` en caso contrario

### Nota importante

Los “comentarios dentro de tag sections” terminan como `ASS.Tag.Unknown` o `ASS.Tag.Junk` (tags internos), **no** como `Section.Comment`.