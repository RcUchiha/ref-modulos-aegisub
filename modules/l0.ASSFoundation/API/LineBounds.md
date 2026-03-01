# LineBounds (l0.ASSFoundation) — v0.5.0

`LineBounds` representa los **bounds visuales reales** (rectángulo mínimo) de una línea ASS, normalmente calculados con **SubInspector**.

Se usa para:
- obtener el `w/h` real de render (incluyendo bordes, blur, etc. según SubInspector),
- tener bounds **por frame** en líneas fbf/animadas,
- comparar resultados (hash por frame) para detectar cambios.

> En ASSFoundation, `LineContents:getLineBounds()` suele delegar el cálculo a SubInspector y devuelve un `LineBounds`.

---

## Estructura (campos)

`LineBounds` guarda:

### `[1]: ASS.Point` (top-left)
Punto mínimo `(xMin, yMin)` del rectángulo global.

### `[2]: ASS.Point` (bottom-right)
Punto máximo `(xMax, yMax)` del rectángulo global.

### `w: number`
Ancho global: `xMax - xMin`.

### `h: number`
Alto global: `yMax - yMin`.

### `fbf: table`
Tabla con bounds por frame (cuando aplica). Incluye:

- `fbf.off`: frame base/offset
- `fbf.n`: número de frames almacenados
- `fbf[frameNumber]`: entry por frame con esta estructura:

```lua
{
  [1] = ASS.Point(x1, y1),   -- top-left
  [2] = ASS.Point(x2, y2),   -- bottom-right
  w = w,
  h = h,
  hash = bound.hash,
  solid = bound.solid
}
```

### Frame sin bound

Si un frame no tiene bound, se guarda como:

```lua
{ w = 0, h = 0, hash = false }
```

---

## `animated: boolean`

Se marca desde `line.si_exhaustive`. Si está activo, el objeto se trata como animado incluso si hay frames sin bound útil.

---

## `rawText: string` (opcional)

Si `keepRawText` es `true`, guarda `line.text` original.

---

## Campos derivados útiles

- `firstHash`: hash del primer frame (`fbf[fbf.off].hash`)
- `firstFrameIsSolid`: `solid` del primer frame (`fbf[fbf.off].solid`)

---

# Constructor

## `LineBounds.new(line, bounds, frames, first?, last?, offset?, keepRawText?) -> LineBounds`

Crea el objeto a partir de resultados de SubInspector.

---

## Parámetros

- `line`: instancia `Line` (se usa `line.si_exhaustive` y opcionalmente `line.text`)
- `bounds`: arreglo de entradas por frame (SubInspector)
  - cada entrada típica trae `x`, `y`, `w`, `h`, `hash`, `solid`
  - puede ser `false` para indicar “sin bound”
- `frames`: arreglo de números de frame correspondiente a cada entrada de `bounds`
- `first` (default `1`): índice inicial dentro de `bounds`
- `last` (default `#bounds`): índice final dentro de `bounds`
- `offset` (default `0`): offset a sumar a los frames al escribir en `fbf`
- `keepRawText`: si es `true`, guarda `rawText = line.text`

---

## Comportamiento

1) Ajusta `first` / `last` / `offset` a defaults.

2) Si hay datos útiles o la línea es `animated`, crea `fbf` con:

```lua
off = frames[first] + offset
n = last - first + 1
```

3) Recorre los frames:

- Si hay bound:
  - calcula `x2 = x1 + w`, `y2 = y1 + h`
  - guarda entry con puntos y `hash` / `solid`
  - acumula min/max global (`x1Min/y1Min/x2Max/y2Max`)

- Si no hay bound:
  - guarda:

```lua
{ w = 0, h = 0, hash = false }
```

4) Si hubo min/max global, define:

```lua
[1] = ASS.Point(x1Min, y1Min)
[2] = ASS.Point(x2Max, y2Max)
```

Luego define `w/h` y define `firstHash` y `firstFrameIsSolid`.

5) Si no hubo nada, deja:

```lua
w = 0
h = 0
fbf = { n = 0 }
```

---

# Comparación

## `equal(other) -> boolean`

Compara si dos `LineBounds` son equivalentes.

### Reglas

- Si `other` no es `LineBounds`, lanza error.
- Si ambos tienen `w == 0` (sin bounds), se consideran iguales.
- Si difieren en:
  - `w`, `h`
  - `animated`
  - `fbf.n` o `fbf.off`
  ⇒ retorna `false`.

Si todo lo anterior coincide, compara `hash` frame a frame:

- si cualquier `hash` difiere ⇒ `false`
- si todos coinciden ⇒ `true`

---

# Ejemplo (uso típico con batch)

Cuando se calculan bounds en lote, `LineBoundsBatch.run()` suele crear un `LineBounds` así:

- SubInspector devuelve `bounds` y `times.frames`.
- Para cada línea se construye:

```lua
ASS.LineBounds(line, bounds, frames, firstIndex, lastIndex, offset)
```

Y se regresa un mapa:

```lua
outBounds[foreignKey] = LineBounds(...)
```

---

# Notas prácticas

- `hash` por frame sirve como comparación rápida: si cambió el render, cambia el hash.
- En líneas animadas (`si_exhaustive`), puedes tener frames sin bound válido y aun así conservar estructura `fbf`.
- `rawText` es útil para depurar cuando SubInspector devuelve error o para cache.