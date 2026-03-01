# Parser.Drawing (l0.ASSFoundation) — v0.5.0

Este parser convierte el string de comandos ASS Drawing (lo que se usa con `\pN`) en:

- una lista de `Contour`s (cada una con comandos `Move`, `Line`, `Bezier`, `Close`),
- y opcionalmente una sección `junk` (como `Section.Comment`) si hay basura o comandos rotos.

Es el núcleo que permite que `Section.Drawing` sea estructurado y editable (no solo un string).

---

# API

## `getContours(str, parent, splitContours=true) -> (contours, junk?)`

Parsea `str` (drawing commands) y devuelve:

- `contours`: `{ASS.Draw.Contour...}`
- `junk`: `ASS.Section.Comment` (opcional) con fragmentos inválidos o trailing junk

---

## Parámetros

- `str`: string de comandos (ej. `"m 0 0 l 10 0 10 10 0 10"`)
- `parent`: referencia al objeto padre (para asignar `contour.parent = parent`)
- `splitContours`:
  - si `true`, cada `m` (Move) inicia un contorno nuevo
  - si `false`, no separa por `Move` (depende del consumidor)

---

# Cómo parsea (resumen realista)

1) “Tokeniza” el string: hace `trim` y divide por espacios.
2) Recorre los tokens:
   - reconoce comandos por el `commandMapping` (`m`, `l`, `b`, `c`, etc.)
   - o detecta comandos implícitos cuando aparecen números (más coordenadas) después de un comando anterior.
3) Para cada comando:
   - lee la cantidad esperada de ordinadas (coords) según la definición del comando
   - intenta convertir cada una a número
   - si faltan coords o hay tokens inválidos:
     - puede fallar con error
     - o intentar “arreglar” dependiendo de `ASS.config.fixDrawings`

---

# Manejo de dibujos rotos (`fixDrawings` + `quirks`)

Cuando hay ordinadas malformadas o “con basura pegada”, el parser decide qué hacer según:

- `ASS.config.fixDrawings`
- `ASS.config.quirks` (`VSFilter` vs `libass`)

---

## `quirks = VSFilter`

- Si un número viene con “junk” al final:
  - intenta rescatar la parte numérica
  - el resto lo mueve a `junk`
- Pero si aparece basura, VSFilter tiende a detener el procesamiento de comandos posteriores.
- El parser imita ese comportamiento y corta el parseo.

---

## `quirks = libass`

- libass es más tolerante:
  - puede ignorar ciertas coordenadas inválidas
  - puede continuar parseando si detecta comandos válidos después
- El parser ajusta la lógica:
  - mueve basura a `junk`
  - ajusta el consumo de parámetros para intentar completar el comando

---

# Salida y estructura

- Cada contorno termina siendo `ASS.Draw.Contour` con comandos en orden.
- `Close` cierra un contorno.
- Si `splitContours` está activo:
  - un `Move` (después del primero) dispara un contorno nuevo.

---

# Notas útiles

- El parser solo crea un comando si:
  - tiene parámetros suficientes, o
  - el comando no requiere parámetros.
- Se usa un constructor interno rápido (`__defNew`) si existe, por rendimiento.
- Si se detecta un comando no soportado, se registra error y continúa (según configuración).