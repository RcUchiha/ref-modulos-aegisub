# Tag.Fade (l0.ASSFoundation) — v0.5.0

`Tag.Fade` representa los tags ASS de fade:

- `\fad(t1,t2)` (fade simple)
- `\fade(a1,a2,a3,t1,t2,t3,t4)` (fade avanzado)

Controla la opacidad de la línea en el tiempo.

---

# Sintaxis ASS

## Fade simple

```ass
{\fad(200,300)}Texto
```

- `t1`: duración de fade-in (ms)
- `t2`: duración de fade-out (ms)

## Fade avanzado

```ass
{\fade(0,255,0,0,200,800,1000)}Texto
```

- `a1,a2,a3`: alpha en 3 fases (0..255 en convención ASS: 0 opaco, 255 transparente)
- `t1..t4`: tiempos (ms, relativos al inicio de la línea)

---

# Representación (estructura interna)

Como tag complejo, normalmente guarda:

## Modo `fad`

- `inMs: number`
- `outMs: number`

## Modo `fade`

- `a1, a2, a3: number`
- `t1, t2, t3, t4: ASS.Time` (o números ms)

Además suele guardar qué modo está usando (`isAdvanced`, `mode`, etc.).

---

# Métodos (típicos)

## `getTagString() -> string`

Serializa en el mismo modo:

- `\fad(in,out)` si es simple
- `\fade(a1,a2,a3,t1,t2,t3,t4)` si es avanzado

---

## `copy() -> Fade`

Copia profunda.

---

## `equal(other) -> boolean`

Compara todos los campos relevantes.

---

# Interacción con alpha

`fad/fade` afecta alpha global visual, pero no necesariamente se modela como `\alpha` permanente.

- En render, `fade` multiplica/combina opacidad por tiempo.
- En términos de estado de tags, se conserva como su propio tag.

---

# Notas prácticas

- `\fad` es un atajo muy común y fácil de generar.
- `\fade` permite fases específicas y es útil para flashes.
- Como afecta el render por tiempo, puede cambiar bounds medidos si el renderer considera alpha para hashing (SubInspector suele hacerlo).

---

# Ejemplos

## Fade simple

```ass
{\fad(200,300)}Texto
```

## Fade avanzado

```ass
{\fade(0,255,0,0,200,800,1000)}Texto
```

---

# Resumen

`Tag.Fade` representa:

- fade simple (`\fad`)
- fade avanzado (`\fade`)

Es un tag complejo ligado a tiempo, y afecta el render de forma dinámica.