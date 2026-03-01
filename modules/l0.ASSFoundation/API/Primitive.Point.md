# Primitive.Point (l0.ASSFoundation) — v0.5.0

`ASS.Point` representa un punto bidimensional `(x, y)`.

Es una estructura base utilizada en:

- `LineBounds`
- `Draw.Contour`
- comandos `Move`, `Line`, `Bezier`
- cálculos geométricos
- clips vectoriales
- transformaciones

Es uno de los primitives fundamentales del sistema.

---

# Constructor

## `ASS.Point(x=0, y=0) -> Point`

Crea un nuevo punto con coordenadas:

- `x: number`
- `y: number`

Si no se pasan valores, ambos default a `0`.

Internamente:

- valida que sean números
- los guarda como propiedades directas del objeto

---

# Propiedades

### `x: number`
Coordenada horizontal.

### `y: number`
Coordenada vertical.

---

# Métodos principales

## `copy() -> Point`

Devuelve una copia del punto.

```lua
local p2 = p1:copy()
```

---

## `equal(other) -> boolean`

Devuelve `true` si:

- `other` es `ASS.Point`
- `self.x == other.x`
- `self.y == other.y`

---

## `add(other) -> Point`

Suma componente a componente:

```lua
(x1 + x2, y1 + y2)
```

Retorna un nuevo `Point`.

---

## `sub(other) -> Point`

Resta componente a componente:

```lua
(x1 - x2, y1 - y2)
```

Retorna un nuevo `Point`.

---

## `scale(sx, sy?) -> Point`

Escala el punto:

- Si solo se pasa `sx`, escala ambos ejes.
- Si se pasa `sx, sy`, escala por eje.

```lua
local p2 = p1:scale(2)
local p3 = p1:scale(2, 3)
```

---

## `distance(other) -> number`

Devuelve la distancia euclidiana entre dos puntos:

```lua
sqrt((x2-x1)^2 + (y2-y1)^2)
```

---

## `length() -> number`

Devuelve la distancia del punto al origen `(0,0)`.

Equivalente a:

```lua
sqrt(x^2 + y^2)
```

---

## `normalize() -> Point`

Devuelve un punto con la misma dirección pero longitud 1.

Si la longitud es 0, puede retornar `(0,0)` según implementación.

---

## `rotate(angleDegrees) -> Point`

Rota el punto alrededor del origen `(0,0)`.

- `angleDegrees` está en grados.
- Conversión interna a radianes.
- Aplica transformación estándar:

```lua
x' = x*cos(a) - y*sin(a)
y' = x*sin(a) + y*cos(a)
```

---

# Uso dentro de Drawing

En `Draw.Contour` y comandos:

- `Move`
- `Line`
- `Bezier`

Cada coordenada suele almacenarse como `ASS.Point`.

Ejemplo conceptual:

```lua
local p = ASS.Point(100, 50)
contour:addPoint(p)
```

---

# Uso en LineBounds

`LineBounds` usa `Point` para almacenar:

- `[1]` → top-left
- `[2]` → bottom-right

Esto permite:

- cálculos geométricos
- comparaciones
- extensiones futuras sin usar simples tablas `{x=..., y=...}`

---

# Notas

- `Point` es inmutable en el sentido conceptual (los métodos retornan nuevos puntos).
- No contiene lógica de render.
- Es una primitiva geométrica pura.
- Se usa intensivamente en Drawing y Bounds.

---

# Ejemplo completo

```lua
local p1 = ASS.Point(10, 20)
local p2 = ASS.Point(5, 5)

local p3 = p1:add(p2)
print(p3.x, p3.y) -- 15, 25

local d = p1:distance(p2)
print(d)
```

---

# Resumen

`ASS.Point` es la base geométrica del sistema.

Sin `Point` no existirían:

- bounds
- contornos
- transformaciones
- clips
- cálculo de distancias
- rotaciones

Es pequeño, pero absolutamente fundamental.