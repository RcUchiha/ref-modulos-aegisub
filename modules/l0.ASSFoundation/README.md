# l0.ASSFoundation

**Versión documentada:** v0.5.0 (canal `release` de DependencyControl)  
**Namespace:** `l0.ASSFoundation`

---

## ¿Qué es?

ASSFoundation es una librería avanzada para manipulación estructurada de líneas ASS.

Permite:

- Parsear una línea en secciones (`Text`, `Tag`, `Drawing`, `Comment`)
- Manipular tags sin usar regex
- Insertar, reemplazar y limpiar tags correctamente
- Calcular tags efectivos
- Dividir líneas en múltiples segmentos
- Obtener métricas y bounds visuales reales

---

## Importación

### Con DependencyControl

```lua
local DependencyControl = require("l0.DependencyControl")

local dep = DependencyControl{
  {
    {"l0.ASSFoundation", version="0.5.0"}
  }
}

local ASS = dep:requireModules()
```

---

## Documentación técnica

La documentación detallada por archivo y clase se encuentra en:

- [API (índice completo)](API.md)