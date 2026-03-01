# Tag.Transform (l0.ASSFoundation) — v0.5.0

`ASS.Tag.Transform` representa el tag ASS:

```
\t(...)
```

Permite animar uno o varios tags en un intervalo de tiempo dentro de una línea.

Es uno de los tags más complejos del sistema porque:

- contiene otros tags internamente
- tiene rango temporal (`t1`, `t2`)
- puede tener aceleración (`accel`)
- interactúa con `TagList.merge()` y `diff()`

---

# Sintaxis ASS

Formatos válidos en ASS:

```
\t(<tags>)
\t(t1,t2,<tags>)
\t(t1,t2,accel,<tags>)
```

Ejemplos:

```
\t(\fs60)
\t(0,500,\alpha&H00&)
\t(0,1000,2,\fscx200\fscy200)
```

---

# Estructura interna

Un `Tag.Transform` contiene:

## `startTime: ASS.Time | nil`
Tiempo inicial relativo al inicio de la línea.

## `endTime: ASS.Time | nil`
Tiempo final relativo al inicio de la línea.

Si ambos son `nil`, el transform aplica a toda la duración de la línea.

---

## `accel: number`
Factor de aceleración.

- `1` = lineal
- `>1` = ease-in
- `<1` = ease-out

---

## `tags: {Tag...}`
Lista de tags animados dentro del transform.

Estos tags:

- no afectan inmediatamente el estado base
- se aplican dinámicamente durante el intervalo

---

# Constructor

## `ASS.Tag.Transform{ ... }`

Se construye normalmente a través del parser (`ASS.getTagFromString`).

Estructura conceptual:

```lua
ASS.Tag.Transform{
  startTime = ASS.Time(...),
  endTime   = ASS.Time(...),
  accel     = 1,
  tags      = { Tag1, Tag2, ... }
}
```

---

# Métodos principales

## `getTagString() -> string`

Serializa el transform nuevamente a formato ASS.

Ejemplo resultado:

```
\t(0,500,1,\fs60\alpha&H00&)
```

---

## `copy() -> Transform`

Devuelve una copia profunda del transform:

- copia tiempos
- copia accel
- copia todos los tags internos

---

## `getDuration(lineDuration?) -> Time`

Devuelve la duración efectiva del transform.

- Si tiene `startTime` y `endTime`, devuelve `endTime - startTime`
- Si no, puede usar la duración completa de la línea (según implementación)

---

# Comportamiento dentro de TagList

Cuando un `Section.Tag` se convierte en `TagList`:

- los `Transform` se almacenan en `self.transforms`
- NO se guardan dentro de `self.tags[name]`

Esto es clave:

`TagList.tags` representa el estado estático.
`TagList.transforms` representa animaciones.

---

# Interacción con merge()

Durante `TagList.merge()`:

- los transforms se concatenan
- no se deduplican
- si un tag animado también aparece como override normal posterior,
  el override normal puede invalidar el efecto del transform previo

---

# Interacción con diff()

`TagList.diff()`:

- conserva transforms
- considera que un transform en `previous` afecta el estado
- puede requerir generar override aunque el valor base coincida

Esto es importante para generar overrides mínimos correctos.

---

# Aplicación conceptual (cómo funciona en runtime)

En tiempo `t`:

1) Se parte del estado base (`TagList.tags`)
2) Se evalúan transforms activos en ese tiempo
3) Para cada tag animado:
   - se calcula interpolación según `accel`
   - se aplica sobre el estado base

Interpolación típica:

```
progress = (t - t1) / (t2 - t1)
progress = progress ^ accel
```

Luego se interpola valor inicial → valor final.

---

# Limitaciones

- No todos los tags son transformables.
- Tags con `__tag.transformable == false` no deberían animarse.
- Multi-tags (ej. karaoke) no suelen estar dentro de transforms.
- Algunos tags (ej. clip vectorial complejo) pueden requerir lógica especial.

---

# Ejemplo completo

```lua
local tr = ASS.Tag.Transform{
  startTime = ASS.Time(0),
  endTime   = ASS.Time(1000),
  accel     = 1,
  tags = {
    ASS.Tag.FontSize{ value = 60 }
  }
}

print(tr:getTagString())
-- \t(0,1000,1,\fs60)
```

---

# Ejemplo ASS real

```
{\fs40\t(0,500,\fs60)\t(500,1000,\fs40)}Texto
```

Flujo:

- 0–500ms → interpola 40 → 60
- 500–1000ms → interpola 60 → 40

---

# Notas importantes

- Los tiempos son relativos al inicio de la línea.
- Si `t1 > t2`, comportamiento depende del renderer (generalmente ignorado).
- `accel` no es easing complejo, solo potencia exponencial.
- Múltiples transforms pueden solaparse.

---

# Resumen

`Tag.Transform` es:

- un contenedor de animaciones
- dependiente de `Primitive.Time`
- gestionado por `TagList`
- aplicado dinámicamente en evaluación por frame

Es uno de los componentes más complejos de ASSFoundation.