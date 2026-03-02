# Tag.Color (l0.ASSFoundation) — v0.5.0

Los tags de color en ASS controlan:

- colores primarios/bordes/sombras (`\c`, `\2c`, `\3c`, `\4c`)
- alpha global y por canal (`\alpha`, `\1a`, `\2a`, `\3a`, `\4a`)

En ASSFoundation, estas variantes se modelan como clases `ASS.Tag.*` que heredan de `Tag.Base`.

Estos tags son cruciales porque `TagList:getCombinedColor()` combina **alpha + color** para construir un color final tipo:

```
&HAABBGGRR&
```

---

# Colores `\cN`

## Tags soportados (comunes)

- `\c`  → color 1 (primary)
- `\2c` → color 2 (secondary)
- `\3c` → color 3 (outline)
- `\4c` → color 4 (shadow)

---

## Representación

Cada color se guarda normalmente como string ASS:

```
&HBBGGRR&
```

Donde:

- `BB` = Blue  
- `GG` = Green  
- `RR` = Red  

> ASS usa formato **BGR**, no RGB.

---

## Ejemplo ASS

```ass
{\c&H00FFFF&}Texto
```

---

# Alpha `\alpha` y `\Na`

## Tags soportados (comunes)

- `\alpha`  → alpha master (aplica a todos)
- `\1a`     → alpha canal 1
- `\2a`     → alpha canal 2
- `\3a`     → alpha canal 3
- `\4a`     → alpha canal 4

---

## Representación

Alpha se guarda como:

```
&HAA&
```

Donde:

- `00` = totalmente opaco  
- `FF` = totalmente transparente  

> En ASS el alpha está invertido respecto a RGBA tradicional.

---

# Constructor (forma conceptual)

Los tags normalmente se construyen vía parser, pero conceptualmente:

```lua
local c1 = ASS.Tag.Color{ value = "&HBBGGRR&" }
local a  = ASS.Tag.Alpha{ value = "&HAA&" }
```

Para canales específicos:

```lua
local c3 = ASS.Tag.Color3{ value = "&H000000&" }
local a3 = ASS.Tag.Alpha3{ value = "&H80&" }
```

> Los nombres exactos de clase pueden variar (`Color`, `Color2`, etc.), pero el modelo es un tag por canal.

---

# Métodos (heredados de Tag.Base)

Estos tags suelen ser simples (solo `value`), por lo que heredan:

- `getTagString()`
- `copy()`
- `get()`
- `set()`
- `equal()`

Ejemplo:

```lua
local t = ASS.Tag.Color{ value = "&H00FFFF&" }
print(t:getTagString()) -- "\c&H00FFFF&"
```

---

# Interacción con TagList

## `TagList:getCombinedColor(num, styleRef) -> string`

Combina el color `\Nc` y alpha `\Na` (o `\alpha`) para producir:

```
&HAABBGGRR&
```

---

## Flujo conceptual

1) Obtiene el color base:
   - primero desde tags (`\Nc`)
   - si no existe, desde `styleRef.colorN`

2) Obtiene alpha:
   - `\Na` si existe
   - si no, usa `\alpha` como fallback
   - si no, usa alpha del estilo

3) Une ambos y devuelve el string final.

Esto se usa en:

- `getStyleTable()`
- mediciones
- render preview
- exportaciones o conversiones

---

# Notas importantes

- ASS usa BGR, no RGB.
- Alpha está invertido (`00` opaco, `FF` transparente).
- `\alpha` actúa como master respecto a `\1a..\4a`.
- En `TagList.merge()`, alpha master puede invalidar children.
- En `diff()`, children deben compararse contra master previo.

---

# Ejemplos

## Cambiar color primario

```ass
{\c&HFF0000&}Texto
```

---

## Cambiar alpha global

```ass
{\alpha&H80&}Texto
```

---

## Cambiar borde y su alpha

```ass
{\3c&H000000&\3a&H40&\bord3}Texto
```

---

## Obtener color combinado (conceptual)

```lua
local tags = sec:getEffectiveTags(true)
local c = tags:getCombinedColor(1, line:getStyleRef())
print(c) -- "&HAABBGGRR&"
```

---

# Resumen

`Tag.Color` cubre:

- colores por canal (`\c`, `\2c`, `\3c`, `\4c`)
- alpha global y por canal (`\alpha`, `\1a..\4a`)

Es esencial para:

- estilo efectivo
- cálculos de render
- combinación alpha + color
- coherencia en merge/diff