# Tag.Move (l0.ASSFoundation) — v0.5.0

`Tag.Move` representa el tag ASS:

- `\move(x1,y1,x2,y2)`
- `\move(x1,y1,x2,y2,t1,t2)`

Sirve para mover el texto (o el bloque dibujado) desde un punto inicial a uno final, opcionalmente dentro de un intervalo de tiempo.

Este tag es especialmente importante porque:

- afecta la posición real en pantalla (obvio),
- hace que los bounds sean **animados**,
- interactúa con `\pos` (y en la práctica lo reemplaza durante el intervalo),
- su intervalo temporal se interpreta relativo al inicio de la línea.

---

# Sintaxis ASS

## Forma corta (sin tiempo)

```ass
{\move(100,200,300,200)}Texto
```

Interpreta el movimiento como aplicado durante toda la duración de la línea (según renderer).

## Forma completa (con tiempo)

```ass
{\move(100,200,300,200,0,500)}Texto
```

- `t1` y `t2` están en milisegundos relativos al inicio de la línea.
- Se interpola de `(x1,y1)` a `(x2,y2)` solo entre `t1..t2`.

---

# Representación (estructura interna)

`Tag.Move` es un tag “complejo”: no es solo `value`.

Campos conceptuales típicos:

- `x1: number`
- `y1: number`
- `x2: number`
- `y2: number`
- `t1: ASS.Time | nil`
- `t2: ASS.Time | nil`

Al serializar, reconstruye la forma:

- sin tiempos si `t1/t2` no existen,
- con tiempos si están definidos.

---

# Constructor (forma conceptual)

Normalmente el parser lo construye, pero a nivel conceptual:

```lua
local mv = ASS.Tag.Move{
  x1 = 100, y1 = 200,
  x2 = 300, y2 = 200,
  t1 = ASS.Time(0),
  t2 = ASS.Time(500)
}
```

> Los nombres exactos de campos pueden variar internamente, pero el modelo es ese: 2 puntos + (opcional) rango temporal.

---

# Métodos (comunes en tags complejos)

Como tag complejo, normalmente implementa/overridea:

- `getTagString()` (serialización)
- `copy()` (copia profunda)
- `equal()` (comparación por todos los campos relevantes)

Ejemplo conceptual:

```lua
print(mv:getTagString())
-- "\move(100,200,300,200,0,500)"
```

---

# Interpretación del movimiento (conceptual)

En un tiempo `t`:

- si NO hay `t1/t2`: se interpola como si fuera toda la línea (según renderer; muchos tratan como 0..duración).
- si hay `t1/t2`:
  - si `t <= t1` → posición = `(x1,y1)`
  - si `t >= t2` → posición = `(x2,y2)`
  - si `t1 < t < t2` → interpolación lineal:

```text
p = (t - t1) / (t2 - t1)
x = x1 + (x2 - x1) * p
y = y1 + (y2 - y1) * p
```

---

# Interacción con otros tags

## `\pos`

- `\pos(x,y)` define una posición fija.
- `\move(...)` define una posición dependiente del tiempo.

En renderers típicos, cuando `\move` está presente, domina el posicionamiento durante el intervalo.

## `\an`

El punto de anclaje definido por `\an` aplica igual: el movimiento desplaza el anchor.

Ejemplo:

```ass
{\an7\move(100,100,300,100)}Texto
```

Moverá la **esquina superior izquierda** del bloque.

## `\org`

`org` define el origen de rotación. Si hay rotación (`\frz` etc.), el movimiento y rotación combinados pueden producir trayectorias visuales que parecen “curvas” por rotación alrededor de `\org`.

---

# Impacto en bounds

Como la posición varía con el tiempo:

- los bounds por frame pueden cambiar,
- `LineBounds` suele terminar con `fbf` significativo cuando hay muestreo frame a frame,
- si el pipeline activa modo exhaustivo (`si_exhaustive`), se capturan bounds por frame.

En resumen: `\move` suele implicar **bounds animados**.

---

# Validación y normalización (reglas típicas)

- `x1,y1,x2,y2` deben ser números.
- si existen `t1,t2`:
  - deben ser números >= 0
  - si `t1 == t2`, el movimiento “salta” (depende de renderer; muchas veces se ve como inmediato)
  - si `t1 > t2`, suele considerarse inválido o ignorado (renderer-dependent)

---

# Ejemplos

## 1) Movimiento simple

```ass
{\move(100,200,300,200)}Texto
```

## 2) Movimiento con intervalo

```ass
{\move(100,200,300,200,0,500)}Texto
```

## 3) Movimiento + alineación

```ass
{\an5\move(320,240,320,100,0,1000)}Texto
```

## 4) Movimiento + rotación (conceptual)

```ass
{\org(320,240)\frz90\move(100,240,540,240,0,1000)}Texto
```

---

# Ejemplo con TagList / parseo (conceptual)

```lua
local contents = line:getContents()
local tags = contents:getEffectiveTags(true)

-- si existe un move, normalmente estará en tags["move"] o como Tag.Move
local mv = tags.tags and tags.tags.move
```

> Nota: El nombre exacto de clave depende del `__tag.name` de la clase (`"move"` típicamente).

---

# Notas prácticas

- `\move` es lineal; si quieres easing real, normalmente se combina con `\t(...)` sobre `\pos` o se simula con múltiples segmentos.
- Con `\an` “incorrecto”, tu movimiento se verá desplazado (anchor equivocado).
- Para mediciones y colisiones reales, usa bounds por frame (SubInspector) si hay `\move`.

---

# Resumen

`Tag.Move` describe un desplazamiento de posición:

- desde `(x1,y1)` hasta `(x2,y2)`
- opcionalmente dentro de `t1..t2`

Es un tag complejo que:

- cambia la posición en el tiempo,
- suele producir bounds animados,
- depende fuertemente de `\an` para interpretarse correctamente,
- y se integra con pipelines que usan SubInspector/`LineBoundsBatch` para medir frame a frame.