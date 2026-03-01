# TagList (l0.ASSFoundation) — v0.5.0

`TagList` representa el **estado efectivo** de tags ASS en un punto de una línea.

A diferencia de `Section.Tag` (que es “lo que hay dentro de `{...}`”), `TagList` es una estructura “de estado” que:

- conserva **solo la última** instancia de la mayoría de tags (por nombre)
- separa **transforms** (`\t(...)`) en una lista aparte
- conserva **multi-tags** (karaoke y desconocidos) como lista, porque **no se pueden colapsar** a una sola instancia
- maneja **resets** (`\r` / `\rEstilo`) y su efecto en el estado local
- permite **merge** / **diff** de estados de tags (muy útil para generar overrides mínimos)

---

## Estructura interna

### `tags: table<string, Tag>`
Mapa `nombreTag -> objeto Tag` con la última instancia “relevante” de cada tag.

- La mayoría de tags se guardan como “último gana”.
- Tags **globales** (según el sistema interno `__tag.global`) se tratan especial: no se sobreescriben si ya existe uno.
- Clips vectoriales (`\clip(m...)` / `\iclip(m...)`) se tratan especial: solo puede existir uno a la vez.

### `transforms: {ASS.Tag.Transform...}`
Lista de transforms. Se guardan aparte porque puede haber múltiples `\t(...)` y no se reducen a un mapa simple.

### `multiTags: {Tag...}`
Lista de tags de “múltiples apariciones” (multi).

En ASSFoundation se usan para:

- karaoke (`\k`, `\kf`, `\ko`) — afectan offset/estado de forma acumulativa
- tags desconocidos — no se puede asumir comportamiento, así que se conservan todos

### `reset: ASS.String | nil`
Tag `\r` / `\rEstilo` si se detectó un reset relevante en el estado.

### `contentRef`
Referencia al contenido padre (`LineContents`/sección) para poder consultar:

- estilo base (`getStyleRef`)
- defaults (`getDefaultTags`)

### Campos adicionales (existen como parte de la clase)
`startTime`, `endTime`, `accel` (usados en algunos flujos con transforms/tiempos).

---

## Constructor

### `TagList.new(tags, contentRef) -> TagList`

Crea un TagList desde:

- `ASS.Section.Tag` (lo típico)
- `TagList` (copia superficial)
- `nil` (vacío)

#### Si el origen es `Section.Tag`
El constructor recorre los tags y:

- detecta `reset` y elimina estado local previo (manteniendo globales)
- separa transforms en `transforms`
- guarda multi-tags en `multiTags`
- guarda solo la primera instancia de tags globales (si ya existía una)
- evita duplicar vector clips (solo uno)
- elimina “children tags” cuando aparece el master (ej. `\alpha` limpia `\1a..\4a` en el estado)
- filtra transforms para eliminar tags transformados que ya fueron sobreescritos por tags “normales” posteriores

> En resumen: deja el estado “limpio” y sin contradicciones obvias.

---

## API

### `get() -> table`
Devuelve un mapa simple con los valores “en crudo”:

```lua
{ name, tag:get() for name, tag in pairs(self.tags) }
```
Útil para depurar.

---

## `checkTransformed(tagName?) -> table | boolean`

Construye un set con los nombres de tags que aparecen dentro de `\t(...)`.

- **Sin argumentos:**  
  Retorna:

  ```lua
  { [tagName] = true, ... }
  ```

- **Con `tagName`:**  
  Retorna `true` o `false` dependiendo de si ese tag aparece transformado.

---

# Merge de estados

## `merge(tagLists, copyTags=true, returnOnly, overrideGlobalTags, expandResets) -> TagList | self`

Combina este `TagList` con uno o varios `TagList` adicionales.

### Parámetros clave

- `tagLists`: un `TagList` o lista `{TagList...}`
- `copyTags`: si `true`, el resultado se copia (no comparte referencias)
- `returnOnly`: si `true`, devuelve el resultado sin mutar `self`
- `overrideGlobalTags`: si `true`, permite que globales sean sobreescritos en el merge
- `expandResets`: si `true`, al ver un reset expande defaults del estilo correspondiente para reconstruir el estado completo

### Reglas importantes

- Si se encuentra `reset`:
  - Si `expandResets` está activo, se recalcula el estado desde defaults del reset.
  - Si no, se descarta estado local previo y se mantienen globales.

- Tags transformables marcan tags transformados previos como “override” y se purgan de transforms anteriores.

- Clips vectoriales se tratan como mutuamente excluyentes (`\clip` vs `\iclip`).

- `multiTags` se concatenan (y se exige que tengan estrategia: karaoke o `nonOverriding`).

---

# Diff de estados

## `diff(previous, returnOnly, ignoreGlobalState) -> TagList | self`

Calcula el “delta” de estado necesario para pasar de `previous` a `self`.

### Parámetros

- `previous` debe ser `TagList`
- `returnOnly`: si `true`, devuelve un `TagList` diff sin mutar `self`
- `ignoreGlobalState`: si `true`, ignora comparación de tags globales

### Reglas relevantes

#### Reseteos

- Un reset solo se conserva si realmente cambia el estado comparado con el “reference”.
- Si el reset es equivalente al estilo actual, puede omitirse.

#### Tags transformados

- Tags transformados en `previous` cuentan como cambio en `self` (porque afectan estado al aplicar).

#### Children / Master

- Children (`\1a`, etc.) deben aparecer en diff si cambian el estado respecto al master en referencia.

#### Clips vectoriales

- Solo se consideran “diferentes” si el anterior no tenía clip vectorial activo.

### Además

- `transforms` no se deduplican: se conservan.
- `multiTags` se conservan: karaoke/unknown se consideran siempre relevantes.

---

# Lectura de parámetros

## `getTagParams(name, asBool, multiValue) -> any`

Obtiene parámetros del tag si existe en `self.tags[name]`.

- `asBool`: si `true`, convierte a booleano (ej. `bold`, `italic`)
- `multiValue`: si `true`, devuelve `{...}` en vez de un solo valor (cuando aplique)

---

# Colores combinados

## `getCombinedColor(num, styleRef) -> string`

Devuelve el string ASS de color combinado `alpha + color`  
(ej. `&HAABBGGRR&`), tomando:

- `\alphaN` (o estilo) para alpha
- `\cN` (o estilo) para color

`num` va de 1 a 4 (`color1`..`color4`).

---

# Estilo efectivo para métricas

## `getStyleTable(styleRef, name?, coerce?) -> table`

Construye una tabla estilo “efectiva” mezclando:

- defaults desde `styleRef` (tabla de Aegisub, `class == "style"`)
- overrides desde el `TagList` (`align`, `fontname`, `fontsize`, colores, etc.)

Incluye campos como:

- `bold`
- `italic`
- `underline`
- `strikeout`
- `scale_x`
- `scale_y`
- `spacing`
- `angle`
- etc.

Además incluye:

- `raw`: una línea `Style: ...` lista para usarse en algunas APIs/herramientas.

> Útil para `aegisub.text_extents` y para conversiones con Yutils.

---

# Filtrado de tags

## `filterTags(tagNames, tagProps, returnOnly, inverseNameMatch, propCheckExempts={"reset"}) -> (included, removed) | (self, removed)`

Filtra tags por nombre y/o por propiedades internas del tag (`__tag`).

### Parámetros

- `tagNames` puede ser:
  - `string` (un tag)
  - `{string...}` (lista)
  - `nil` (usa todos)

- `tagProps` (opcional): tabla con propiedades esperadas a comparar contra `tag.__tag`

- `inverseNameMatch`: si `true`, invierte el set seleccionado

- `returnOnly`: si `true`, no muta `self` y devuelve `TagList` resultantes

### Comportamientos especiales

- `reset` se maneja aparte (`included.reset` / `removed.reset`)
- `transform` es un “nombre virtual” que apunta a `transforms`
- Por defecto, ciertas props no se comparan para algunos tags (ej. `reset`)

---

# Utilidades

## `isEmpty() -> boolean`

Devuelve `true` si el `TagList` no tiene:

- `tags`
- `reset`
- `transforms`
- `multiTags`

---

## `getGlobal(includeRectClips) -> table`

Devuelve un mapa `name -> tag` con tags globales:

- Si `includeRectClips` es `true`, incluye también rectangular clips como “globalOrRectClip”.
- Si no, solo aquellos con `__tag.global == true`.

---

# Ejemplos

## 1) Construir TagList desde una sección Tag

```lua
local tags = TagList(secTag)
```

---

## 2) Merge de estado (sin mutar)

```lua
local merged = tags:merge(otherTags, true, true)
```

---

## 3) Calcular diff (generar overrides mínimos)

```lua
local diff = current:diff(previous, true)
-- diff ahora contiene solo lo necesario para pasar de previous -> current
```

---

## 4) Filtrar solo tags transformables (ejemplo conceptual)

```lua
local included, removed = tags:filterTags(nil, { transformable = true }, true)
```

---

## 5) Obtener style table efectivo (para métricas)

```lua
local styleTbl = tags:getStyleTable(lineStyle)
local w, h = aegisub.text_extents(styleTbl, "Hola")
```