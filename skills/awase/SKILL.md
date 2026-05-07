---
description: Skill de entrenamiento adaptativo para desarrolladores. Genera 1-2 ejercicios técnicos breves basados en el código de la sesión y mantiene un perfil personal con spaced repetition (SM-2). Úsala cuando el usuario escriba '/awase', '/awase status', '/awase skip', '/awase reset' o '/awase --tipo <tipo>'.
---

# awase — Skill de entrenamiento adaptativo

Esta skill se activa cuando el usuario escribe `/awase` (con o sin argumentos).
Genera ejercicios técnicos breves basados en el código producido en la sesión actual
y mantiene un perfil personal con spaced repetition (algoritmo SM-2).

---

## Subcomandos

| Invocación | Comportamiento |
|---|---|
| `/awase` | Flujo normal: el agente elige tipo de ejercicio |
| `/awase --tipo compare` | Fuerza tipo comparación |
| `/awase --tipo completar` | Fuerza tipo completar snippet |
| `/awase --tipo bug` | Fuerza tipo encontrar bug |
| `/awase --tipo explicar` | Fuerza tipo explicar código |
| `/awase status` | Muestra el perfil del dev |
| `/awase skip` | Omite la sesión sin penalizar el perfil |
| `/awase reset` | Resetea el perfil (pide confirmación) |

---

## Perfil personal

**Ruta:** `~/.awase/profile.json`

Si el fichero no existe, créalo antes de continuar con esta estructura inicial:

```json
{
  "version": "1",
  "created_at": "<hoy ISO 8601>",
  "last_session": null,
  "concepts": {},
  "sessions": []
}
```

Usa siempre las herramientas `Read` y `Write` para leer y escribir el perfil.
Nunca menciones la ruta del fichero ni el JSON directamente al usuario; usa lenguaje natural.

---

## Flujo normal

### Paso 1 — Leer el perfil
Lee `~/.awase/profile.json`. Créalo si no existe.

### Paso 2 — Analizar el código de la sesión
Examina el código producido en esta sesión. Identifica conceptos técnicos relevantes:
patrones, APIs, funciones de librería, estructuras de datos, técnicas de diseño, etc.
Descarta conceptos triviales o demasiado básicos para el nivel del dev.

### Paso 3 — Seleccionar concepto a entrenar
Prioridad:

1. Conceptos de la sesión con `next_review <= hoy` en el perfil (repaso pendiente)
2. Conceptos de la sesión que no están en el perfil (nuevo)
3. Cualquier concepto del perfil con `next_review <= hoy` (repaso genérico)

Si no hay ningún concepto disponible, indícaselo al usuario con una línea breve.

### Paso 4 — Elegir tipo de ejercicio
Si el usuario forzó un tipo con `--tipo`, úsalo.

Si no, elige según el historial del concepto:

| Estado del concepto | Tipos preferidos |
|---|---|
| Nuevo (sin historial) | `completar`, `compare` |
| 1-2 repeticiones correctas | `bug`, `explicar` |
| 3+ repeticiones correctas | cualquiera, preferir `explicar` |

Nunca repitas el mismo tipo que `last_exercise_type` para ese concepto.

### Paso 5 — Generar el ejercicio

Usa código real de la sesión siempre que sea posible.
El ejercicio completo debe poder resolverse en menos de 2 minutos.

**`compare`** — dos versiones de código, el usuario elige la más adecuada y explica por qué:

```
**Ejercicio — [concepto]**

¿Cuál es más adecuada para [contexto breve]?

A)
[código A]

B)
[código B]
```

**`completar`** — snippet con huecos `___` que el usuario debe rellenar:

```
**Ejercicio — [concepto]**

Completa el siguiente código:

[código con ___]
```

**`bug`** — snippet con un bug sutil; el usuario debe encontrarlo y explicarlo:

```
**Ejercicio — [concepto]**

Hay un bug en este código. ¿Lo encuentras?

[código con bug]
```

**`explicar`** — bloque de código; el usuario explica qué hace y por qué:

```
**Ejercicio — [concepto]**

Explica qué hace este código y por qué está escrito así:

[código]
```

### Paso 6 — Esperar la respuesta del usuario
Presenta el ejercicio y espera. No avances hasta recibir una respuesta.

### Paso 7 — Evaluar la respuesta

Mapea la calidad de la respuesta a la escala SM-2:

| q | Criterio |
|---|---|
| 5 | Correcto con explicación sólida |
| 4 | Correcto |
| 3 | Correcto pero con dudas o explicación incompleta |
| 1 | Incorrecto pero el usuario reconoce el error tras el feedback |
| 0 | Incorrecto sin comprensión |

### Paso 8 — Aplicar SM-2 y actualizar perfil

```
si q < 3:
    interval = 1
    repetitions = 0
si q >= 3:
    si repetitions == 0: interval = 1
    si repetitions == 1: interval = 6
    si repetitions > 1: interval = round(interval * easiness_factor)
    easiness_factor = max(1.3, easiness_factor + 0.1 - (5-q) * (0.08 + (5-q) * 0.02))
    repetitions += 1

next_review = hoy + interval días
```

Valores por defecto para un concepto nuevo: `easiness_factor=2.5`, `interval=1`, `repetitions=0`.

Actualiza el concepto en `concepts` y añade una entrada al array `sessions` con:
```json
{
  "date": "<hoy>",
  "exercises": 1,
  "correct": <1 si q>=3, 0 si no>,
  "concepts_trained": ["<concept_id>"]
}
```

Si ya hay una entrada para hoy en `sessions`, acumula en lugar de añadir una nueva.

Actualiza `last_session` con la fecha de hoy.

Escribe el perfil con `Write`.

### Paso 9 — Mostrar feedback

Muestra en tres partes:
1. `✓ Correcto.` o `✗ Incorrecto.` seguido de la explicación del concepto (máximo 3 líneas)
2. Si fue incorrecto, una línea con la respuesta correcta
3. `Próxima revisión de [concepto] en N días.`

---

## Subcomando: status

Lee el perfil y muestra:

```
**Perfil awase**

Conceptos entrenados: N
Sesiones totales: N
Tasa de acierto global: N%

**Próximas revisiones:**
[concepto]        vence [fecha]    intervalo actual: N días

**Conceptos nuevos disponibles en esta sesión:**
[concepto]
```

Si el perfil está vacío, indica que todavía no hay datos.

---

## Subcomando: skip

Responde con una sola línea: `Sesión omitida. El perfil no ha cambiado.`
No leas ni escribas el perfil.

---

## Subcomando: reset

Pide confirmación explícita antes de actuar:

```
¿Seguro que quieres borrar todo el perfil? Escribe "confirmar" para continuar.
```

Si el usuario confirma, elimina `~/.awase/profile.json` usando `Bash` (`rm ~/.awase/profile.json`)
y responde: `Perfil reseteado.`

Si el usuario no confirma o responde otra cosa, responde: `Cancelado.`

---

## Reglas generales

- Un ejercicio debe resolverse en menos de 2 minutos. Si necesitas más contexto, acórtalos.
- Usa siempre código real de la sesión cuando sea posible; solo inventa código como último recurso.
- El feedback es siempre conciso: máximo 3 líneas de explicación.
- No entrenes conceptos triviales (p. ej. sintaxis básica que el dev domina claramente).
- No menciones el fichero JSON ni el algoritmo SM-2 al usuario; habla de "perfil" y "próxima revisión".
- Si la sesión no tiene código relevante, indícalo brevemente y ofrece `/awase skip`.
