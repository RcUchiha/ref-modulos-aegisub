# Tag.ClipVect (l0.ASSFoundation) — v0.5.0

`Tag.ClipVect` representa los clips vectoriales ASS:

- `\clip(m ... )`
- `\iclip(m ... )`
- y su variante con escala:
  - `\clip(scale, m ... )`
  - `\iclip(scale, m ... )`

A diferencia de `ClipRect`, aquí el recorte se define mediante un **drawing path** (comandos `m/l/b/c`).

---

# Sintaxis ASS

## Sin escala

```ass
{\clip(m 0 0 l 100 0 100 50 0 50)}Texto
```

## Con escala

```ass
{\clip(2,m 0 0 l 100 0 100 50 0 50)}Texto
```

## Invertido

```ass
{\iclip(m 0 0 l 100 0 100 50 0 50)}Texto
```

---

# Representación (estructura interna)

Como tag complejo, normalmente guarda:

- `inverse: boolean`  
  `false` para `\clip`, `true` para `\iclip`

- `scale: number | nil`  
  Si existe, corresponde al primer parámetro numérico.

- `drawing: ASS.Draw.DrawingBase | string`  
  El path del clip, normalmente parseado a estructura de drawing.

> En ASSFoundation es común que el drawing se transforme a estructura para poder editarlo y manipularlo geométricamente.

---

# Constructor (forma conceptual)

Normalmente lo crea el parser, pero conceptualmente:

```lua
local cv = ASS.Tag.ClipVect{
  inverse = false,
  scale = 2,
  drawing = "m 0 0 l 100 0 100 50 0 50"
}
```

Para invertido:

```lua
local icv = ASS.Tag.ClipVect{
  inverse = true,
  drawing = "m 0 0 l 100 0 100 50 0 50"
}
```

---

# Métodos (típicos en tags complejos)

## `getTagString() -> string`

Serializa:

- `\clip(drawing)` / `\iclip(drawing)` si no hay `scale`
- `\clip(scale,drawing)` / `\iclip(scale,drawing)` si hay `scale`

---

## `copy() -> ClipVect`

Copia profunda del tag:

- `inverse`
- `scale`
- drawing (si está estructurado, copia el `DrawingBase`)

---

## `equal(other) -> boolean`

Compara:

- `inverse`
- `scale`
- representación del drawing (string o estructura equivalente)

---

# Estado efectivo (TagList)

Conceptualmente, en una línea ASS solo puede existir **un clip activo** a la vez:

- clip rectangular (`ClipRect`)
- clip vectorial (`ClipVect`)
- normal o invertido

En el modelo de estado efectivo (`TagList`), el clip representa una única entidad activa.

> Si múltiples clips aparecen en una misma línea, el comportamiento final depende del renderer.  
> Para resultados predecibles, se recomienda usar un solo clip por línea.

---

# Relación con Parser.Drawing

El contenido vectorial se parsea con el mismo sistema que `Section.Drawing`:

- tokenización
- contornos
- comandos `Move/Line/Bezier/Close`
- manejo de `junk`

Esto permite:

- editar la geometría del clip
- transformar el path
- reutilizar utilidades de dibujo ya existentes

---

# Impacto en bounds

Al igual que `ClipRect`, el clip vectorial puede modificar los bounds medidos por herramientas como SubInspector, ya que recorta el área visible del render.

En clips complejos (paths irregulares), los bounds resultantes pueden diferir del rectángulo envolvente del drawing original, dependiendo del área efectivamente visible.

---

# Casos comunes

## Clip vectorial básico

```ass
{\clip(m 0 0 l 200 0 200 200 0 200)}Texto
```

## Clip vectorial con escala

```ass
{\clip(2,m 0 0 l 200 0 200 200 0 200)}Texto
```

La escala afecta la interpretación del path.

---

## Clip invertido

```ass
{\iclip(m 0 0 l 200 0 200 200 0 200)}Texto
```

Mantiene visible el área fuera del path.

---

# Diferencia clave respecto a ClipRect

- `ClipRect` usa coordenadas simples.
- `ClipVect` usa un drawing completo.
- Ambos representan un único estado de clip activo.
- Ambos pueden afectar bounds medidos.

---

# Resumen

`Tag.ClipVect` modela clips por path:

- puede ser normal o invertido
- puede tener escala
- reutiliza el sistema de drawing
- representa un único clip activo en el estado efectivo
- afecta el render visible y los bounds medidos