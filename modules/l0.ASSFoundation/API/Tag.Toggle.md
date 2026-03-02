# Tag.Toggle (l0.ASSFoundation) — v0.5.0

`Tag.Toggle` agrupa tags booleanos de tipo “toggle”:

- `\b` (bold)
- `\i` (italic)
- `\u` (underline)
- `\s` (strikeout)

En ASS pueden aparecer como:

- `\b1` / `\b0`
- o incluso `\b` (según renderer; muchos lo interpretan como “encender” o “usar default”)

En ASSFoundation suelen modelarse como tags simples con `value` boolean/numérico.

---

# Sintaxis ASS

```ass
{\b1\i1\u1\s0}Texto
```

---

# Representación

## `value: boolean | number`

Dependiendo del tag:

- 1/0
- true/false

En utilidades como `TagList:getTagParams(name, asBool=true)` suele convertirse a booleano.

---

# Métodos (heredados)

- `getTagString()`
- `copy()`
- `get()` / `set()`
- `equal()`

Ejemplo:

```lua
local b = ASS.Tag.Bold{ value = 1 }
print(b:getTagString()) -- "\b1"
```

---

# Interacción con estilos

Estos toggles normalmente sobrescriben:

- `style.bold`
- `style.italic`
- `style.underline`
- `style.strikeout`

Dentro de `TagList:getStyleTable()`.

---

# Notas prácticas

- En ASS, `\b` sin valor puede comportarse distinto según renderer; para scripts serios, siempre serializa explícito (`0/1`).
- Muchos pipelines usan `getTagParams(asBool=true)` para no lidiar con enteros.

---

# Resumen

`Tag.Toggle` cubre toggles booleanos de estilo, altamente usados y clave para métricas y render.