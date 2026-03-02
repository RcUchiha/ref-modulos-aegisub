# Tag.Indexed (l0.ASSFoundation) — v0.5.0

`Tag.Indexed` agrupa tags que tienen un índice explícito en su nombre, por ejemplo:

- `\1c..\4c` (colores)
- `\1a..\4a` (alpha)
- (y otros que pueden aparecer como “canales”)

En ASSFoundation, este tipo de tags suele compartir una base común:

- un índice `num` (1..4)
- un valor asociado (color/alpha/etc.)

---

# Representación

Campos conceptuales:

- `num: number` (1..4)
- `value: any` (según el tag: string, number, etc.)

Ejemplo conceptual:

```lua
local c3 = ASS.Tag.Color{ num = 3, value = "&H000000&" }
```

---

# Serialización

Se serializa como:

- `\{num}{assName}...`

Ejemplos:

- `\3c&H000000&`
- `\2a&H80&`

---

# Interacción con master/children

Muchos indexed tienen master equivalente:

- `\alpha` como master de `\1a..\4a`
- `\c` como master de `\1c..\4c` (según interpretación)

Por eso `TagList` puede aplicar reglas de children/master.

---

# Resumen

`Tag.Indexed` modela tags por canal (1..4):

- guarda índice
- serializa correctamente
- participa en reglas master/children en estado efectivo