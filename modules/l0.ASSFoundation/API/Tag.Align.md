# Tag.Align (l0.ASSFoundation) — v0.5.0

`Tag.Align` representa el tag ASS:

- `\an1` .. `\an9`

Controla la **alineación del texto** dentro del frame, y afecta directamente:

- cómo se posiciona el texto (`\pos`, `\move`)
- cómo se interpreta `\org`
- el cálculo de bounds (porque el anchor cambia)
- conversiones a shape (ej. `Section.Text:getShape()`)

En ASSFoundation este tag suele modelarse como un tag simple con `value` numérico.

---

# Sintaxis ASS

```ass
{\an7}Arriba izquierda
{\an5}Centro
{\an2}Abajo centro
```

Valores de alineación:

```
7 8 9
4 5 6
1 2 3
```

---

# Representación

## `value: number`

Número entero del 1 al 9.

Ejemplo conceptual:

```lua
local an = ASS.Tag.Align{ value = 7 }
```

---

# Métodos (heredados de Tag.Base)

Como tag simple, normalmente hereda:

- `getTagString()`
- `copy()`
- `get()`
- `set()`
- `equal()`

Ejemplo:

```lua
local t = ASS.Tag.Align{ value = 7 }
print(t:getTagString()) -- "\an7"
```

---

# Interacción con TagList

## Dentro de `getStyleTable()`

`TagList:getStyleTable(styleRef)` incorpora `\an` al estilo efectivo, típicamente como:

- `align`
- `alignment`

(según el formato que espere el consumidor).

Esto es esencial para:

- `aegisub.text_extents(styleTbl, text)`
- construcción de fuentes con Yutils
- medición coherente con el render real

---

# Relación con posicionamiento

`\an` define el **punto de anclaje** (anchor) del texto.

Ejemplos:

- `\an7`: el `\pos(x,y)` se interpreta como **esquina superior izquierda** del texto.
- `\an5`: `\pos(x,y)` es el **centro** del texto.
- `\an2`: `\pos(x,y)` es el **centro inferior**.

Esto implica que para el mismo `(x,y)`:

- cambiar `\an` cambia la ubicación visual final
- cambian los bounds
- cambian los clips
- cambian posibles colisiones

---

# Uso con shapes (`Section.Text:getShape()`)

Cuando conviertes texto a shape, el flujo típico es:

1) Se obtienen tags efectivos (`getEffectiveTags`)
2) Se resuelve alineación efectiva
3) Se calcula offset del shape según anchor
4) Se ajusta posición para que coincida con el render ASS real

Si ignoras `\an`, el shape quedará desplazado.

---

# Validación y normalización

Reglas típicas:

- `value` debe ser entero
- rango permitido: `1..9`
- valores fuera de rango → error o clamp (según implementación)

---

# Ejemplos

## Alinear arriba izquierda

```ass
{\an7\pos(100,100)}Texto
```

---

## Alinear centro

```ass
{\an5\pos(320,240)}Texto
```

---

## Alinear abajo centro

```ass
{\an2\pos(320,450)}Subtítulo
```

---

# Ejemplo con TagList (conceptual)

```lua
local tags = sec:getEffectiveTags(true)
local styleTbl = tags:getStyleTable(line:getStyleRef())

local w, h = aegisub.text_extents(styleTbl, "Hola")
```

`styleTbl` ya incluye la alineación efectiva.

---

# Notas prácticas

- `\an` es uno de los tags que más rompe scripts cuando se ignora.
- Bounds calculados sin considerar `\an` suelen quedar corridos.
- Siempre resuelve el align efectivo antes de:
  - calcular offsets
  - generar clips
  - convertir a drawing
  - hacer colisiones

---

# Resumen

`Tag.Align` (`\an1..9`) define el anchor del texto.

Afecta:

- posición real en pantalla
- bounds
- conversiones a drawing
- mediciones con `getStyleTable`

Es un tag simple en estructura, pero crítico en comportamiento.