# Section.Tag (l0.ASSFoundation) — v0.5.0

Representa una sección de **tags ASS** (lo que va dentro de `{...}`) dentro de una línea parseada (`LineContents`).

Internamente mantiene una lista `tags` con objetos tag (`ASS.Tag.*`) y ofrece API para:

- crear/parsear secciones Tag desde strings, TagList o tags sueltos
- iterar/modificar tags por nombre y por rango
- insertar/eliminar tags
- serializar a string (`\tag...`)
- calcular tags efectivos en ese punto (incluyendo defaults y secciones previas)

---

## Propiedades

### `tags: {Tag...}`
Arreglo con los tags contenidos en la sección, en el orden en que existen/serán serializados.

> Importante: cada tag mantiene referencia `tag.parent = thisSection` cuando está dentro de la sección.

---

## Alias

### `getStyleTable(name?) -> table`
`Section.Tag` reutiliza el mismo `getStyleTable` que `Section.Text`.

---

## Constructor

### `Section.Tag.new(tags?, transformableOnly?, tagSortOrder=ASS.tagSortOrder) -> Section.Tag`

Crea una sección Tag desde distintos tipos de entrada:

- **`nil`** → crea una sección vacía.
- **`ASS.TagList`** → crea la sección respetando:
  - `reset` si existe
  - orden definido por `tagSortOrder` (y luego transforms/multiTags)
  - si `transformableOnly` es `true`, filtra tags no transformables (salvo `Tag.Unknown`)
- **`Section.Tag`** → copia tags desde otra sección (mantiene `parent` del original como referencia base).
- **`string`** (o `{ "string" }`) → parsea tags crudos usando el parser de ASSFoundation.
- **`{Tag...}`** → lista de objetos Tag; valida que sean tags soportados.

---

## Iteración y modificación

### `callback(callbackFn, tagNames?, first=1, last?, relative?, reverse?) -> number|false`

Iterador/modificador de tags (similar a `LineContents:callback`, pero a nivel tag).

- `tagNames` puede ser:
  - `nil` → recorre todos los tags
  - `string` → un nombre de tag
  - `table` → lista de nombres de tags

**Rangos:**
- `first` / `last` deben ser enteros.
- Ambos deben ser >0 o ambos <0.
- Soporta índices negativos.
- Si `relative` es `true`, el rango se aplica por **conteo de coincidencias**, no por índice absoluto.
- `reverse` invierte la dirección de iteración (útil con negativos o rangos relativos).

**Firma del callback:**
`callbackFn(tag, tagsArray, iAbs, iMatch, toRemoveTable)`

**Retornos del callback:**
- `false` → marca el tag para eliminar
- `nil` / `true` → deja el tag igual
- cualquier otro valor → reemplaza el tag por el valor retornado (y se le asigna `parent`)

**Retorno de `callback`:**
- `number` → cuántos tags procesó
- `false` → si no procesó ninguno

---

### `modTags(tagNames, callbackFn, first?, last?, relative?) -> number|false`

Atajo: llama a `callback(...)` con parámetros reordenados para uso común.

---

### `getTags(tagNames?, first?, last?, relative?) -> {Tag...}`

Recolecta tags que coincidan con `tagNames` (o todos si es `nil`) y retorna una lista.

---

## Eliminación

### `remove() -> Section.Tag|{removed...}`

- Si esta sección no tiene `parent` (no está dentro de un `LineContents`), retorna la propia sección.
- Si tiene `parent`, delega a `parent:removeSections(thisSection)` para sacarla de la línea.

---

### `removeTags(tags?, first?, last?, relative?) -> ({removed...}, matchedCount)`

Elimina tags dentro de la sección.

Casos comunes:

- **Sin parámetros**: elimina todos los tags (`tags` queda vacío).
- **`removeTags(first, last)`** (sin `tags` y sin `relative`): elimina por rango.
- **`tags` como string / Tag / lista**:
  - strings se interpretan como nombres de tag (normalizados internamente)
  - Tag objects se comparan por identidad del objeto

**Notas:**
- soporta rango negativo
- soporta modo `relative` (por coincidencias)
- retorna:
  - `removed`: lista con tags removidos
  - `matchedCount`: cuántos coincidieron (aunque no todos necesariamente se removieran si hay rango relativo)

---

## Inserción

### `insertTags(tags, index?) -> Tag|{Tag...}`

Inserta uno o varios tags en la sección.

- `index` debe ser entero y **no puede ser 0**.
- Si `index` no se pasa, se inserta al final (con lógica interna para no romper cuando la sección está vacía).
- Acepta como `tags`:
  - `Tag` (objeto individual)
  - `{Tag...}` (lista)
  - `Section.Tag`
  - `ASS.TagList` (se convierte a `Section.Tag(tagList)` y se insertan sus `tags`)

**Validaciones:**
- cada elemento debe ser un Tag válido
- el nombre del tag debe existir en el mapa de tags (si no, error)
- el tipo/clase del tag debe coincidir con el tipo esperado para ese nombre (si no, error)

**Retorno:**
- si insertas 1 → retorna ese tag insertado
- si insertas varios → retorna lista con los insertados (en el orden insertado)

---

### `insertDefaultTags(tagNames, index?) -> Tag|{Tag...}`

Inserta tags por defecto derivados del estilo de la línea.

- `tagNames` puede ser:
  - `string` (un tag)
  - `{string...}` (lista)
- obtiene defaults con `parent:getDefaultTags()`
- inserta usando `insertTags(...)`

---

## Serialización

### `getString() -> string`

Devuelve el string concatenado de todos los tags:

- llama `tag:getTagString()` por cada tag
- concatena y retorna

> Nota: esto devuelve el contenido **sin** envolver en `{}`; el envoltorio lo hace el serializador de línea/sección según corresponda.

---

## Tags efectivos

### `getEffectiveTags(includeDefault, includePrevious=true, copyTags=true) -> TagList`

Calcula el estado de tags efectivo en este punto:

1) Si `includeDefault` es `true`, parte de `parent:getDefaultTags(...)`.
2) Si `includePrevious` es `true` y existe `prevSection`, mezcla tags efectivos previos.
3) Construye un `TagList` con los tags de **esta sección**.
4) Devuelve el merge final.

Parámetros:
- `includeDefault`: incluir tags por defecto del estilo
- `includePrevious`: incluir acumulación previa (secciones anteriores)
- `copyTags`: si `true`, trabaja con copias (más seguro si vas a mutar)

---

## Ejemplos

### 1) Obtener todos los tags `\bord` en la sección
```lua
local tags = secTag:getTags("border")
```

---

### 2) Eliminar todos los tags `\blur`

```lua
secTag:removeTags("blur")
```

---

### 3) Insertar un tag en la posición 1

```lua
secTag:insertTags(ASS.Tag.Border{ value = 3 }, 1)
```

---

### 4) Modificar todos los tags `\fs` (font size)

```lua
secTag:modTags("font_size", function(tag)
  tag.value = tag.value + 4
  return tag
end)
```

---

### 5) Serializar tags

```lua
local s = secTag:getString()
-- ejemplo: "\bord3\blur0.6\fs54"
```