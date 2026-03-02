# Tag.Transform (l0.ASSFoundation) — v0.5.0

`ASS.Tag.Transform` representa el tag ASS:

```
\t(...)
```

Permite animar uno o varios tags dentro de un intervalo de tiempo relativo al inicio de la línea.

Es uno de los tags más complejos del sistema porque:

- contiene otros tags internamente
- tiene rango temporal (`t1`, `t2`)
- puede tener aceleración (`accel`)
- interactúa con `TagList.merge()` y `TagList.diff()`

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
- `>1` = ease-in (curva más lenta al inicio)
- `<1` = ease-out (curva más rápida al inicio)

El valor se interpreta como potencia exponencial sobre el progreso temporal.

---

## `tags: {Tag...}`

Lista de tags animados dentro del transform.

Estos tags:

- no modifican inmediatamente el estado base
- representan valores objetivo que el renderer interpolará en el intervalo

---

# Constructor

## `ASS.Tag.Transform{ ... }`

Se construye normalmente a través del parser (`ASS.getTagFromString`).

Estructura conceptual:

```lua
ASS.Tag.Transform{
  startTime = ASS.Time(0),
  endTime   = ASS.Time(1000),
  accel     = 1,
  tags      = {
    ASS.Tag.FontSize{ value = 60 }
  }
}
```

---

# Métodos principales

## `getTagString() -> string`

Serializa el transform nuevamente a formato ASS.

Ejemplo resultado:

```
\t(0,1000,1,\fs60)
```

---

## `copy() -> Transform`

Devuelve una copia profunda del transform:

- copia tiempos
- copia accel
- copia todos los tags internos

---

## `equal(other) -> boolean`

Compara:

- tiempos
- accel
- lista de tags internos

---

# Comportamiento dentro de TagList

Cuando un `Section.Tag` se convierte en `TagList`:

- los `Transform` se almacenan en `self.transforms`
- NO se guardan dentro de `self.tags[name]`

Esto es clave:

- `TagList.tags` representa el estado estático base.
- `TagList.transforms` representa animaciones declarativas.

ASSFoundation **no ejecuta la animación**; solo modela su estructura.

---

# Interacción con `merge()`

Durante `TagList.merge()`:

- los transforms se concatenan
- no se deduplican automáticamente
- se preservan en el orden en que aparecen

La resolución final en tiempo depende del renderer.

---

# Interacción con `diff()`

`TagList.diff()`:

- conserva transforms
- considera que un transform modifica el comportamiento dinámico
- no los colapsa dentro del estado estático

Esto es importante para generar overrides mínimos correctos.

---

# Aplicación conceptual (en el renderer)

En tiempo `t`, el renderer típicamente:

1) Parte del estado base (`TagList.tags`)
2) Evalúa transforms activos en ese instante
3) Para cada tag animado:
   - calcula progreso:
     ```
     progress = (t - t1) / (t2 - t1)
     progress = progress ^ accel
     ```
   - interpola valor inicial → valor objetivo

> Esta interpolación la realiza el renderer (libass/VSFilter), no ASSFoundation.

---

# Limitaciones y consideraciones

- No todos los tags son transformables.
- Tags con `__tag.transformable == false` no deberían animarse.
- Multi-tags (ej. karaoke) normalmente no se usan dentro de transforms.
- Tags complejos (ej. clips vectoriales) pueden requerir soporte específico del renderer.
- Múltiples transforms pueden solaparse.

---

# Ejemplo ASS real

```
{\fs40\t(0,500,\fs60)\t(500,1000,\fs40)}Texto
```

Conceptualmente:

- 0–500ms → interpola 40 → 60
- 500–1000ms → interpola 60 → 40

---

# Resumen

`Tag.Transform` es:

- un contenedor declarativo de animaciones
- dependiente de `Primitive.Time`
- gestionado por `TagList`
- evaluado dinámicamente por el renderer

Es uno de los componentes más complejos del sistema de tags ASS.