# Line (l0.ASSFoundation) — v0.5.0

`Line` representa una línea ASS estructurada dentro de ASSFoundation.

No es solo un string de diálogo: es un objeto que:

- mantiene referencia al estilo
- almacena tiempos
- contiene el texto original
- permite parseo estructurado (`LineContents`)
- interactúa con SubInspector
- controla modo exhaustivo (`si_exhaustive`)

---

# Propósito

`Line` es la unidad base sobre la que operan:

- `LineContents`
- `TagList`
- `LineBounds`
- `LineBoundsBatch`
- Parser
- Secciones (`Section.*`)

---

# Estructura principal

## Campos comunes

### `text: string`
Texto ASS original de la línea (incluye overrides `{...}`).

### `startTime: number`
Tiempo de inicio en milisegundos.

### `endTime: number`
Tiempo de fin en milisegundos.

### `styleRef`
Referencia al estilo asociado (tabla tipo Aegisub `class == "style"`).

### `si_exhaustive: boolean`
Indica si la línea debe procesarse en modo exhaustivo (frame por frame).

---

# Constructor

## `Line.new(lineTable, styleRef?) -> Line`

Crea una instancia `Line`.

### Parámetros

- `lineTable`: tabla estilo Aegisub (`class == "dialogue"`)
- `styleRef` (opcional): tabla de estilo ya resuelta

Si no se pasa `styleRef`, el objeto puede resolverlo desde contexto superior.

---

# Métodos principales

## `getStyleRef() -> table`

Devuelve la referencia de estilo efectiva asociada a la línea.

Se usa para:

- generar defaults
- construir `TagList`
- medición con `aegisub.text_extents`
- construcción de fuentes con Yutils

---

## `getDefaultTags() -> TagList`

Construye un `TagList` basado únicamente en el estilo.

Incluye:

- fontname
- fontsize
- colores
- align
- scale
- bold/italic/underline/strikeout
- etc.

Este `TagList` se usa como base para:

- `Section.Tag:getEffectiveTags()`
- merges
- resets (`\r`)

---

## `getContents() -> LineContents`

Parsea el texto y devuelve un objeto `LineContents`.

Internamente:

- divide en secciones (`Text`, `Tag`, `Drawing`, `Comment`)
- enlaza `prevSection` / `nextSection`
- mantiene referencia al `Line` padre

---

## `getLineBounds(options?) -> LineBounds`

Calcula bounds para la línea.

Normalmente:

- delega a `LineBoundsBatch`
- invoca SubInspector
- retorna un objeto `LineBounds`

Si `si_exhaustive == true`, puede calcular frame por frame.

---

## `copy() -> Line`

Devuelve una copia del objeto `Line`.

Usado cuando:

- se quieren modificar tags sin afectar original
- se necesita generar variantes
- se trabaja en pipelines de transformación

---

# Relación con LineContents

`Line` no manipula directamente los tags internos.

Para eso:

```lua
local contents = line:getContents()
```

Luego:

- `contents:replaceTags(...)`
- `contents:getEffectiveTags()`
- `contents:callback(...)`
- etc.

---

# Relación con TagList

El flujo típico:

```lua
local contents = line:getContents()
local tags = contents:getEffectiveTags(true)
```

O para defaults:

```lua
local defaults = line:getDefaultTags()
```

---

# Relación con LineBounds

```lua
local bounds = line:getLineBounds()
```

Luego puedes:

```lua
print(bounds.w, bounds.h)
```

O comparar:

```lua
if bounds1:equal(bounds2) then
  print("Render idéntico")
end
```

---

# Modo exhaustivo

Si:

```lua
line.si_exhaustive = true
```

Entonces:

- SubInspector calcula todos los frames
- `LineBounds` guarda estructura `fbf`
- la línea se considera animada aunque algunos frames no tengan bound

Útil para:

- líneas con `\t(...)`
- animaciones complejas
- validación frame a frame

---

# Flujo típico completo

```lua
local line = Line(dialogueRow, styleRef)

local contents = line:getContents()
local tags = contents:getEffectiveTags(true)

local bounds = line:getLineBounds()

print(bounds.w, bounds.h)
```

---

# Notas importantes

- `Line` es el punto de entrada estructurado a ASSFoundation.
- No debes manipular el texto crudo directamente si quieres comportamiento consistente.
- Usa siempre `LineContents` para trabajar con secciones.
- Usa `TagList` para estados efectivos.
- Usa `LineBounds` para medición real.

---

# Resumen

`Line` conecta:

- datos ASS crudos
- estilo
- parser estructurado
- sistema de tags
- medición real vía SubInspector

Es la pieza central del flujo de trabajo en ASSFoundation.