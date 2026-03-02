# Tag.Weight (l0.ASSFoundation) — v0.5.0

`Tag.Weight` se refiere al control de grosor de fuente y su relación con bold.

En ASS el grosor se puede expresar como:

- `\b1/\b0` (bold “clásico”)
- y en algunos contextos existe manejo de `weight` como valor numérico (dependiendo del sistema de tags de ASSFoundation)

En ASSFoundation, `Weight` suele representar un peso numérico (estilo moderno), mientras `Toggle` cubre el bold 0/1.

---

# Representación

## `value: number`

Peso típico:

- 400 = normal
- 700 = bold

> Nota: el soporte real depende del renderer y del tag específico; en ASS estándar clásico, el bold es un boolean.

---

# Interacción con `getStyleTable()`

Si está presente, puede mapear a:

- `bold` (boolean) si el renderer solo soporta toggle
- o `weight` si hay campo específico

---

# Notas prácticas

- Si tu flujo final es VSFilter/libass, no asumas que un `weight` numérico se respeta como CSS.
- Úsalo principalmente para normalización interna y generación consistente de estilos.

---

# Resumen

`Tag.Weight` modela un peso numérico de fuente cuando el sistema lo soporta.