# API — l0.ASSFoundation (v0.5.0)

> Fuente: canal `release` del feed de ASSFoundation (DependencyControl).  
> Objetivo: documentar cada función/método público por archivo/clase.

---

## Entry point
### `.moon`
- Exports / tabla principal
- Cómo se instancia / cómo se usa `ASS\parse` (si existe aquí)

---

## Núcleo
### `Base.moon`
- Clases base / utilidades comunes

### `ClassFactory.lua`
- Constructor del sistema de clases (herencia/metadatos)

### `FoundationMethods.moon`
- Métodos “de conveniencia” agregados a objetos

### `LineContents.moon`
- Parseo y manipulación de contenido de línea
- `commit`, `callback`, limpieza, reemplazo de tags, etc.

### `TagList.moon`
- Manejo de tags, overrides, transforms, diff/merge, etc.

### `LineBounds.moon`
- Cálculo de bounds (texto/drawings)

### `LineBoundsBatch.moon`
- Procesamiento masivo con SubInspector

---

## Parser
### `Parser/Sections.moon`
### `Parser/LineText.moon`
### `Parser/Drawing.moon`

---

## Primitive
### `Primitive/Number.moon`
### `Primitive/Point.moon`
### `Primitive/String.moon`
### `Primitive/Time.moon`

---

## Section
### `Section/Text.moon`
### `Section/Tag.moon`
### `Section/Drawing.moon`
### `Section/Comment.moon`

---

## Draw
### `Draw/DrawingBase.moon`
### `Draw/Contour.moon`
### `Draw/CommandBase.moon`
### `Draw/Move.moon`
### `Draw/MoveNc.moon`
### `Draw/Line.moon`
### `Draw/Bezier.moon`
### `Draw/Close.moon`

---

## Tag
### `Tag/Base.moon`
### `Tag/Align.moon`
### `Tag/Color.moon`
### `Tag/Fade.moon`
### `Tag/Move.moon`
### `Tag/Transform.moon`
### `Tag/ClipRect.moon`
### `Tag/ClipVect.moon`
### `Tag/String.moon`
### `Tag/Toggle.moon`
### `Tag/Indexed.moon`
### `Tag/Weight.moon`
### `Tag/Unknown.moon`
