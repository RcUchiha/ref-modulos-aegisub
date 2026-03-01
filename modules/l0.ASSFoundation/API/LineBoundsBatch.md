# LineBoundsBatch (l0.ASSFoundation) — v0.5.0

`LineBoundsBatch` es el componente encargado de calcular **bounds en lote** usando SubInspector.

Mientras `LineBounds` representa el resultado de una sola línea,  
`LineBoundsBatch`:

- organiza múltiples líneas
- invoca SubInspector una sola vez
- distribuye los resultados
- construye objetos `LineBounds`
- devuelve un mapa de resultados por clave

---

## Objetivo

Permite calcular bounds para muchas líneas de forma eficiente, evitando:

- llamadas repetidas a SubInspector
- reprocesamiento innecesario
- pérdida de sincronización entre líneas animadas

---

# Flujo general

1) Se reciben líneas (`Line` / `LineContents`).
2) Se preparan datos para SubInspector.
3) Se ejecuta SubInspector en modo batch.
4) Se reciben `bounds` y `frames`.
5) Se construye un `LineBounds` por línea.
6) Se devuelve un mapa con resultados.

---

# Método principal

## `LineBoundsBatch.run(lines, options?) -> table`

Ejecuta el cálculo batch.

### Parámetros

- `lines`: colección de líneas (`Line` objects)
- `options` (opcional):
  - puede incluir flags como modo exhaustivo, frame ranges, offset, etc.

---

# Resultado

Devuelve un mapa:

```lua
outBounds[foreignKey] = LineBounds(...)
```

Donde:

- `foreignKey` identifica cada línea original
- el valor es una instancia `LineBounds`

---

# Cómo interactúa con SubInspector

SubInspector devuelve típicamente:

- `bounds` → arreglo de resultados por frame
- `times.frames` → arreglo con números de frame

Cada entrada de `bounds` contiene:

```lua
{
  x = number,
  y = number,
  w = number,
  h = number,
  hash = number|string,
  solid = boolean
}
```

O puede ser:

```lua
false
```

si no hubo bound para ese frame.

---

# Construcción de LineBounds

Para cada línea procesada:

```lua
ASS.LineBounds(line, bounds, frames, firstIndex, lastIndex, offset)
```

El constructor:

- calcula min/max global
- construye estructura `fbf`
- asigna hashes por frame
- determina si la línea es animada
- opcionalmente guarda `rawText`

---

# Modo exhaustivo

Si la línea tiene:

```lua
line.si_exhaustive == true
```

El cálculo:

- procesa todos los frames
- conserva estructura `fbf` incluso si algunos frames no tienen bound
- marca el objeto como `animated`

Esto es útil para:

- animaciones complejas
- líneas con transforms
- detección precisa de cambios por frame

---

# Offset y rangos

`LineBoundsBatch` permite controlar:

- `firstIndex`
- `lastIndex`
- `offset`

Esto permite:

- calcular solo un rango de frames
- desplazar numeración
- sincronizar bounds con otras líneas

---

# Uso típico

```lua
local results = LineBoundsBatch.run(lines)

local lb = results[lineKey]

if lb and lb.w > 0 then
  print("Width:", lb.w)
end
```

---

# Comparación entre líneas

Gracias al `hash` por frame, puedes hacer:

```lua
if lb1:equal(lb2) then
  print("Render idéntico")
end
```

Sin necesidad de recalcular geometría.

---

# Casos prácticos

- Detectar si una línea animada cambió visualmente.
- Determinar área máxima ocupada por un grupo de líneas.
- Precalcular bounds antes de aplicar máscaras o efectos.
- Validar si un efecto alteró realmente el render.

---

# Notas técnicas

- `LineBoundsBatch` no calcula geometría por sí mismo: delega completamente a SubInspector.
- La estructura `fbf` permite almacenar resultados por frame sin perder información global.
- El uso de `hash` hace que comparaciones sean rápidas y eficientes.
- Es recomendable reutilizar resultados si no cambió el texto o los tags.

---

# Relación con otros módulos

- `LineBounds`
- `LineContents:getLineBounds()`
- `TagList` (porque los tags afectan el render)
- SubInspector (motor externo de medición)

---

# Resumen

`LineBoundsBatch` es el puente entre:

- texto ASS estructurado
- motor de render (SubInspector)
- representación estructurada (`LineBounds`)

Permite mediciones masivas eficientes y comparaciones fiables entre estados de render.