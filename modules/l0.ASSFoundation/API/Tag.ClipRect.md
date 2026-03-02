# Tag.ClipRect (l0.ASSFoundation) — v0.5.0

`Tag.ClipRect` representa los tags ASS de clip rectangular:

- `\clip(x1,y1,x2,y2)`
- `\iclip(x1,y1,x2,y2)`

Un clip rectangular recorta el render a un rectángulo.

- `\clip` **mantiene** lo que está dentro del rectángulo.
- `\iclip` **invierte** el clip (mantiene lo que está fuera).

---

# Sintaxis ASS

```ass
{\clip(100,100,300,200)}Texto
{\iclip(100,100,300,200)}Texto
```

---

# Representación (estructura interna)

Como tag complejo, normalmente guarda:

- `inverse: boolean`  
  `false` para `\clip`, `true` para `\iclip`

- `x1, y1, x2, y2: number`  
  Coordenadas del rectángulo.

> En ASS, el rectángulo se define mediante dos esquinas opuestas (`x1,y1` y `x2,y2`), no mediante `w/h`.

---

# Constructor (forma conceptual)

Normalmente lo crea el parser, pero conceptualmente:

```lua
local c = ASS.Tag.ClipRect{
  inverse = false,
  x1 = 100, y1 = 100,
  x2 = 300, y2 = 200
}
```

Para `\iclip`:

```lua
local ic = ASS.Tag.ClipRect{
  inverse = true,
  x1 = 100, y1 = 100,
  x2 = 300, y2 = 200
}
```

---

# Métodos (típicos en tags complejos)

## `getTagString() -> string`

Serializa de regreso a ASS:

- `\clip(x1,y1,x2,y2)` si `inverse == false`
- `\iclip(x1,y1,x2,y2)` si `inverse == true`

---

## `copy() -> ClipRect`

Copia profunda del tag (coordenadas + `inverse`).

---

## `equal(other) -> boolean`

Compara:

- `inverse`
- coordenadas

---

# Estado efectivo (TagList)

Conceptualmente, en una línea ASS solo puede existir **un clip activo** a la vez:

- clip rectangular (`ClipRect`)
- clip vectorial (`ClipVect`)
- normal o invertido

En el modelo de estado efectivo (`TagList`), el clip representa una única entidad activa.

> Si múltiples clips aparecen en una misma línea, el comportamiento final depende del renderer.  
> Para resultados predecibles, se recomienda usar un solo clip por línea.

---

# Interacción con `TagList.getGlobal()`

En algunos flujos, los clips rectangulares pueden tratarse como:

- globales especiales
- o como categoría `globalOrRectClip`

Dependiendo del parámetro:

```lua
tags:getGlobal(true)
```

Si `includeRectClips == true`, los rectangulares pueden incluirse en el mapa global.

---

# Relación con bounds

Un clip **no cambia** el layout interno del texto (fuente, tamaño, etc.),  
pero sí puede cambiar:

- el área renderizada visible
- los bounds reales medidos por herramientas como SubInspector

Ejemplo:

```ass
{\clip(0,0,50,50)\bord10}Texto
```

Aunque el texto sea grande, el bound medido puede quedar reducido al área visible.

---

# Casos comunes

## Clip normal

```ass
{\clip(100,100,300,200)}Texto
```

Visible solo dentro del rectángulo.

---

## Clip invertido

```ass
{\iclip(100,100,300,200)}Texto
```

Visible fuera del rectángulo.

---

# Notas prácticas

- Los clips no se acumulan ni se intersectan automáticamente.
- Solo puede existir un clip activo en el estado efectivo.
- Evita múltiples clips en una misma línea.
- Un clip puede reducir bounds medidos visualmente.
- En pipelines con SubInspector, los clips afectan el hash por frame.

---

# Resumen

`Tag.ClipRect` modela `\clip(...)` y `\iclip(...)` en forma rectangular:

- guarda coordenadas
- guarda si es invertido
- representa un único estado de clip activo
- puede redefinirse por overrides posteriores
- afecta el render visible y los bounds medidos