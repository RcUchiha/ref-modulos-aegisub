# Tag.Base (l0.ASSFoundation) — v0.5.0

`ASS.Tag.Base` es la clase base de todos los tags ASS.

Todos los tags concretos (`Align`, `Color`, `Move`, `Transform`, etc.) heredan de esta clase.

Define:

- comportamiento común
- serialización
- copia
- metadatos internos (`__tag`)
- validación y normalización básica

---

# Estructura interna

## `__tag: table`

Cada clase de tag tiene una definición interna `__tag` que describe su comportamiento.

Campos típicos dentro de `__tag`:

- `name`: nombre lógico del tag (ej. `"font_size"`)
- `assName`: nombre ASS (ej. `"fs"`)
- `global`: boolean (si el tag es global)
- `transformable`: boolean (si puede usarse dentro de `\t`)
- `multi`: boolean (si puede repetirse múltiples veces)
- `reset`: boolean (si es `\r`)
- `type`: tipo esperado de valor
- `children`: lista de tags hijos (ej. `alpha` vs `1a..4a`)
- `master`: referencia al tag padre (si aplica)

Este metadata controla cómo el tag interactúa con:

- `TagList`
- `merge()`
- `diff()`
- `filterTags()`
- `checkTransformed()`

---

# Propiedades comunes

## `value`

La mayoría de tags tienen una propiedad `value`.

Ejemplo:

```lua
ASS.Tag.FontSize{ value = 50 }
```

No todos los tags usan solo `value` (ej. `Move`, `Clip`, `Transform` tienen estructuras más complejas).

---

## `parent`

Referencia a la `Section.Tag` que contiene el tag.

Se asigna automáticamente al insertar el tag en una sección.

---

# Constructor

Normalmente no se instancia `Tag.Base` directamente.

Cada subclase define su propio constructor:

```lua
ASS.Tag.FontSize{ value = 40 }
```

Internamente:

- valida tipos
- normaliza valores
- asigna `__tag`
- puede aplicar coerciones

---

# Métodos principales

## `getTagString() -> string`

Serializa el tag a formato ASS.

Ejemplo:

```lua
local t = ASS.Tag.FontSize{ value = 40 }
print(t:getTagString())
-- "\fs40"
```

Cada subclase implementa su versión específica.

---

## `copy() -> Tag`

Devuelve una copia profunda del tag.

- copia `value`
- copia estructuras internas si existen
- NO copia `parent`

---

## `get() -> any`

Devuelve el valor interno del tag.

Para tags simples:

```lua
local size = tag:get()
```

---

## `set(value)`

Actualiza el valor del tag (si aplica).

Puede validar tipo y rango.

---

## `equal(other) -> boolean`

Devuelve `true` si:

- ambos son del mismo tipo
- tienen el mismo valor
- coinciden propiedades relevantes

---

# Tags simples vs complejos

## Tags simples

Ejemplo:

- `FontSize`
- `Bold`
- `Italic`
- `Border`

Solo tienen `value`.

---

## Tags complejos

Ejemplo:

- `Move`
- `ClipVect`
- `Transform`
- `Fade`

Contienen múltiples valores internos.

Estos redefinen:

- `getTagString()`
- `copy()`
- validación

---

# Children y Master

Algunos tags tienen jerarquía.

Ejemplo:

- `\alpha` (master)
- `\1a`, `\2a`, `\3a`, `\4a` (children)

Reglas:

- Si aparece el master, los children pueden invalidarse.
- En `diff()`, children deben evaluarse respecto al master del estado anterior.

Esto se maneja usando:

```
__tag.master
__tag.children
```

---

# Global vs Local

Algunos tags son `global`.

Ejemplo típico:

- `\pos`
- `\org`

En `TagList.merge()`:

- tags globales no se sobreescriben fácilmente
- pueden requerir bandera `overrideGlobalTags`

---

# Transformable

`__tag.transformable` indica si el tag puede usarse dentro de `\t(...)`.

Si `false`, debería ignorarse dentro de transforms.

---

# Multi-tags

`__tag.multi == true` indica que el tag:

- puede aparecer múltiples veces
- no se colapsa a "último gana"

Ejemplo:

- karaoke
- tags desconocidos

---

# Reset

`__tag.reset == true` identifica el tag `\r`.

Este tag:

- borra estado local previo
- puede cargar estilo específico (`\rEstilo`)
- afecta cómo `TagList` construye el estado

---

# Validación y normalización

Cada subclase puede:

- convertir valores a número
- forzar rangos
- normalizar formatos (ej. colores `&HBBGGRR&`)

La lógica vive en cada clase concreta, no en `Tag.Base`.

---

# Ejemplo simple

```lua
local t = ASS.Tag.Border{ value = 3 }

print(t.__tag.name)        -- "border"
print(t:getTagString())    -- "\bord3"
```

---

# Resumen

`Tag.Base` define:

- el contrato común de todos los tags
- cómo se serializan
- cómo se copian
- cómo interactúan con `TagList`
- cómo se clasifican (global, multi, transformable, reset)

Es la base del sistema completo de overrides ASS en ASSFoundation.