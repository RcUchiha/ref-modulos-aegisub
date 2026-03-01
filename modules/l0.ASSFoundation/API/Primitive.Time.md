# Primitive.Time (l0.ASSFoundation) — v0.5.0

`ASS.Time` representa un tiempo/duración y provee utilidades para convertir entre:

- milisegundos
- frames (cuando hay FPS)
- timestamps tipo ASS (`h:mm:ss.cs`)
- rangos/clamps

Se usa mucho en:

- transforms `\t(...)` (tiempos y aceleración)
- cálculos de duración de línea
- batch de bounds por frame (cuando el flujo requiere mapear tiempo→frame)

---

# Constructor

## `ASS.Time(value=0, unit?) -> Time`

Crea un objeto `Time`.

- `value`: número (por defecto `0`)
- `unit` (opcional): indica cómo interpretar `value`

Según la implementación, el constructor suele normalizar a una unidad base (típicamente **milisegundos**) y guardar el valor interno en un campo único.

> Nota: en ASSFoundation, el tiempo casi siempre se trata como milisegundos para ser compatible con Aegisub.

---

# Representación

## `ms: number`

Valor interno en milisegundos (o equivalente).

> Si el objeto usa otro nombre interno, en la práctica debes tratarlo como “tiempo normalizado”.

---

# Métodos principales

## `copy() -> Time`

Devuelve una copia del tiempo.

```lua
local t2 = t1:copy()
```

---

## `get() -> number`

Devuelve el valor numérico base (usualmente ms).

---

## `set(value, unit?) -> self`

Asigna un nuevo valor, con el mismo esquema de unidad del constructor.

---

## `add(other) -> Time`

Suma dos tiempos (normalizando unidades si aplica) y devuelve un nuevo `Time`.

```lua
local t3 = t1:add(t2)
```

---

## `sub(other) -> Time`

Resta dos tiempos y devuelve un nuevo `Time`.

---

## `mul(k) -> Time`

Multiplica por escalar.

---

## `div(k) -> Time`

Divide por escalar.

---

## `clamp(minTime, maxTime) -> Time`

Devuelve el tiempo limitado al rango `[minTime, maxTime]`.

---

# Conversión a formato ASS

## `toAssTimestamp() -> string`

Convierte el tiempo a timestamp estilo ASS:

```
h:mm:ss.cs
```

Donde `cs` son centésimas.

### Ejemplo

```lua
local t = ASS.Time(12345) -- 12.345s
print(t:toAssTimestamp()) -- "0:00:12.34"
```

> En ASS, el timestamp tradicional usa centésimas; puede haber redondeo.

---

# Conversión desde ASS

## `fromAssTimestamp(str) -> Time`

Convierte un string `h:mm:ss.cs` a `Time`.

### Ejemplo

```lua
local t = ASS.Time.fromAssTimestamp("0:01:02.50")
print(t:get()) -- ~62500 ms
```

---

# Conversión a frames

## `toFrames(fps, roundMode?) -> number`

Convierte el tiempo a número de frames.

- `fps`: frames por segundo (number)
- `roundMode` (opcional):
  - `"floor"`
  - `"ceil"`
  - `"round"`

### Ejemplo

```lua
local f = ASS.Time(1000):toFrames(23.976, "round")
```

---

## `fromFrames(frame, fps) -> Time`

Crea un `Time` desde un número de frame y FPS.

```lua
local t = ASS.Time.fromFrames(240, 24)
```

---

# Uso típico con líneas

Duración de una línea:

```lua
local dur = ASS.Time(line.endTime):sub(ASS.Time(line.startTime))
print(dur:get())
```

---

# Uso típico con transforms `\t(...)`

La mayoría de transforms se expresan como:

- `t1` y `t2` dentro del override
- `accel` como factor aparte

`ASS.Time` facilita:

- normalizar `t1/t2`
- clamping al rango de la línea
- convertir a frames si necesitas un muestreo por frame

---

# Notas prácticas

- Si mezclas FPS flotante (23.976) con redondeo, documenta qué modo usas (`floor`/`round`) para evitar drift.
- Para `\t`, los tiempos se interpretan relativos al inicio de la línea (en ASS).
- En Aegisub, `start_time` y `end_time` están en ms: `Time` encaja perfecto.

---

# Ejemplo completo

```lua
local t1 = ASS.Time(500)      -- 0.5s
local t2 = ASS.Time(1500)     -- 1.5s
local t3 = t2:sub(t1)         -- 1.0s

print(t3:get())               -- 1000
print(t3:toAssTimestamp())    -- "0:00:01.00"

local f = t3:toFrames(24, "round")
print(f)                      -- ~24
```

---

# Resumen

`ASS.Time` te da un tipo de tiempo consistente para:

- aritmética (sumas/restas)
- clamp de rangos
- timestamps ASS
- conversión tiempo↔frames

Es especialmente importante cuando documentes `Tag.Transform` (`\t(...)`) y cualquier sistema que dependa de muestreo por frame.