# Tag.Unknown (l0.ASSFoundation) — v0.5.0

`Tag.Unknown` representa un tag que:

- empieza con `\`
- pero no existe en el mapa conocido de tags de ASSFoundation
- o no pudo parsearse correctamente

Se conserva para no perder información al:

- parsear
- modificar
- re-serializar

---

# Ejemplos típicos

- tags personalizados de otros scripts
- errores tipográficos
- tags no soportados por el parser en esa versión

Ejemplo:

```ass
{\miTagRaro(123)\fs40}Texto
```

`miTagRaro` se conservará como `Unknown`.

---

# Representación

Campos conceptuales comunes:

- `name: string` (nombre del tag sin `\`)
- `raw: string` (representación original)
- `value: string` o parámetros crudos (según implementación)

---

# Serialización

`getTagString()` suele devolver el string original (o reconstruido) para no romper compatibilidad.

---

# Interacción con TagList

En muchos flujos:

- `Unknown` se conserva como `multiTag` (no se colapsa),
- o se deja fuera del estado efectivo estático,
- pero siempre se re-serializa para no perderlo.

---

# Notas prácticas

- No intentes “interpretar” Unknown a menos que tengas una definición real del tag.
- Para scripts, lo correcto es preservarlo y no romper el texto.

---

# Resumen

`Tag.Unknown` existe para conservar tags no reconocidos sin perder información al round-trip.