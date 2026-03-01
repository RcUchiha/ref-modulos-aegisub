# Section.Text (l0.ASSFoundation) — v0.5.0

Representa un tramo de **texto plano** dentro de una línea parseada (`LineContents`), es decir: lo que está *fuera* de `{...}` y *fuera* de dibujos `\pN`.

Esta sección modela contenido textual puro y proporciona utilidades para:

- Medición de texto
- Cálculo de tags efectivos
- Conversión a dibujo (shape)
- Inserción de tags por carácter
- División del texto
- Limpieza (trim)

---

## Propiedades

### `value: string`

Contenido textual de la sección.

### `len: number` (propiedad calculada)

Longitud del texto en **unidades Unicode** (no bytes).

Internamente se calcula como:

```lua
unicode.len(value)
```

---

## Constructor

### `Section.Text.new(value="") -> Section.Text`

Crea una nueva sección de texto.

- Valida que `value` sea string.
- Normaliza el valor recibido.
- Inicializa referencias internas (`parent`, `prevSection`, etc. si aplica).

---

## Serialización

### `getString() -> string`

Devuelve el contenido textual tal cual (`value`).

No agrega llaves `{}` ni procesa tags.

---

## Tags efectivos / estilo

### `getEffectiveTags(includeDefault, includePrevious=true, copyTags=true) -> TagList`

Calcula los tags efectivos en este punto del contenido.

Parámetros:

- `includeDefault` → si es `true`, incluye los tags por defecto derivados del estilo.
- `includePrevious` → si es `true`, acumula tags efectivos de secciones anteriores.
- `copyTags` → si es `true`, retorna copia del `TagList`.

Retorna un `TagList` representando el estado completo de tags aplicables en este punto.

---

### `getStyleTable(name?) -> table`

Devuelve una tabla de estilo derivada de los tags efectivos.

Se usa principalmente para:

- Medición de texto
- Construcción de fuentes con Yutils
- Cálculo de métricas

---

## Medición de texto

### `getTextExtents() -> (w, h, descent, extlead)`

Mide el texto usando:

```lua
aegisub.text_extents(styleTable, value)
```

Devuelve:

- `w` → ancho
- `h` → alto
- `descent`
- `extlead`

---

### `getTextMetrics(calculateBounds) -> (metrics, tagList, shape?)`

Requiere **Yutils**.

Proceso:

1. Construye objeto de fuente con `getYutilsFont()`.
2. Obtiene métricas reales del texto.
3. Si `calculateBounds` es `true`, genera shape y calcula bounds.

Devuelve:

- `metrics.width`
- `metrics.height`
- `tagList`
- opcionalmente `shape`

> Si Yutils no está disponible, lanza error.

---

## Conversión a dibujo (shape)

### `getShape(applyRotation=false) -> ASS.Draw.DrawingBase`

Convierte el texto a shape usando Yutils:

- Genera contornos vectoriales.
- Ajusta posición según alineación efectiva.
- Si `applyRotation` es `true`, aplica rotación según tag de ángulo.
- Devuelve `ASS.Draw.DrawingBase`.

---

### `convertToDrawing(applyRotation) -> Section.Drawing`

Convierte esta sección en una `Section.Drawing`.

Proceso interno:

1. Genera shape.
2. Borra `value`.
3. Asigna `contours`.
4. Ajusta `scale`.
5. Cambia el metatable a `ASS.Section.Drawing`.

La sección deja de comportarse como texto.

---

### `expand(x, y) -> Section.Drawing`

Atajo que:

1. Convierte a drawing.
2. Ejecuta `expand` del lado de `Section.Drawing`.

---

## Fuente Yutils

### `getYutilsFont() -> (fontObj, tagList)`

Requiere **Yutils**.

Construye fuente usando:

- `fontname`
- `bold`
- `italic`
- `underline`
- `strikeout`
- `fontsize`
- `scale_x`
- `scale_y`
- `spacing`

Internamente usa:

```lua
Yutils.decode.create_font(...)
```

---

## Split / inserción de tags

### `splitAtChar(index, mutate) -> (leftTextSection, rightTextSection)`

Divide el texto en dos secciones en una posición de carácter.

Características:

- Soporta índice negativo:

  ```lua
  unicode.len(value) + index + 1
  ```

- Si `mutate` es `true`, reutiliza la sección actual como lado izquierdo.
- Siempre crea una nueva `Section.Text` para el lado derecho.

---

### `insertTagsAtChar(index, tags)`

Inserta tags en una posición específica del texto.

- Valida que `tags` sea:
  - `Section.Tag`
  - `TagList`
  - lista de `Tag`
- En caso contrario, lanza error.
- Internamente crea o reutiliza una `Section.Tag`.

Este método depende del comportamiento de `Section.Tag` y `LineContents`.

---

## Limpieza (trim)

### `trimLeft()`

Aplica:

```lua
string.trimLeft(value)
```

---

### `trimRight()`

Aplica:

```lua
string.trimRight(value)
```

---

### `trim() -> Section.Text`

Aplica:

```lua
string.trim(value)
```

Devuelve la propia instancia.

---

## Ejemplos

### Medir texto

```lua
local w, h = sec:getTextExtents()
```

---

### Dividir texto por carácter

```lua
local left, right = sec:splitAtChar(5, false)
```

---

### Convertir texto a drawing

```lua
-- requiere Yutils
local drawingSec = sec:convertToDrawing(true)
```