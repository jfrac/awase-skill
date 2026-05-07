# awase

> `/awase` — adaptive developer training skill for Claude Code

Skill de entrenamiento adaptativo para desarrolladores que trabajan con agentes de IA. Se invoca en mitad de cualquier sesión de Claude Code y genera 1-2 ejercicios breves basados en el código que el agente acaba de producir.

La idea es simple: el agente ya sabe qué código escribiste. En lugar de ignorarlo, `/awase` lo convierte en material de entrenamiento personalizado para que no pierdas habilidades técnicas por delegación.

---

## Cómo funciona

1. Trabajas con Claude Code normalmente
2. En cualquier momento escribes `/awase`
3. El agente analiza el código reciente de la sesión
4. Genera 1-2 ejercicios cortos (< 2 minutos en total)
5. Da feedback inmediato
6. Actualiza tu perfil personal con el resultado

Los ejercicios pueden ser de cuatro tipos: **comparar** dos opciones de código, **completar** un snippet, **encontrar un bug**, o **explicar** qué hace un bloque. El agente elige el tipo más adecuado según tu historial.

## Spaced Repetition

El sistema usa el algoritmo **SM-2** (el mismo que Anki) para decidir qué entrenar. Cada concepto técnico tiene un intervalo de revisión que se alarga si aciertas y se acorta si fallas. Con el tiempo, el agente sabe exactamente qué necesitas repasar y cuándo.

El perfil es **personal**: se guarda en `~/.awase/profile.json` en tu máquina, no en el repo.

---

## Instalación

### Opción A — Skill personal (más simple)

Copia el SKILL.md a tu directorio de skills global:

```bash
mkdir -p ~/.claude/skills/awase
curl -fsSL https://raw.githubusercontent.com/jfrac/awase-skill/main/skills/awase/SKILL.md \
  > ~/.claude/skills/awase/SKILL.md
```

### Opción B — Plugin via marketplace

```
/plugin marketplace add jfrac/awase-skill
/plugin install awase@jfrac/awase-skill
```

En ambos casos, `/awase` queda disponible en **todos tus proyectos** de forma inmediata.

### El perfil se crea automáticamente

La primera vez que ejecutes `/awase`, el agente crea `~/.awase/profile.json` automáticamente. No necesitas hacer nada más.

Opcional: añade `~/.awase/` a tu `.gitignore` global para asegurarte de que el perfil nunca se commitea.

```bash
echo "~/.awase/" >> ~/.gitignore_global
git config --global core.excludesfile ~/.gitignore_global
```

---

## Uso

```
/awase                    flujo normal, el agente decide
/awase --tipo compare     fuerza ejercicio de comparación
/awase --tipo completar   fuerza ejercicio de completar snippet
/awase --tipo bug         fuerza ejercicio de encontrar bug
/awase --tipo explicar    fuerza ejercicio de explicar código
/awase status             muestra tu perfil: conceptos, tasas de acierto, próximas revisiones
/awase skip               omite la sesión sin penalizar el perfil
/awase reset              resetea el perfil completo (pide confirmación)
```

---

## Ejemplo de sesión

```
> /awase

**Ejercicio — `Promise.all` vs `Promise.allSettled`**

¿Cuál es más adecuada para el `processUsers` que acabamos de escribir?

A)
const results = await Promise.all(ids.map(fetchUser));

B)
const results = await Promise.allSettled(ids.map(fetchUser));

---

> B, porque fetchUser puede fallar y no quiero que cancele el resto

✓ Correcto. `Promise.allSettled` espera a todas las promesas independientemente
de si fallan, ideal cuando quieres resultados parciales en lugar de un fallo total.

Perfil actualizado. Próxima revisión de `Promise.allSettled` en 6 días.
```

---

## Estructura del repo

```
awase-skill/
  .claude-plugin/
    plugin.json           manifiesto del plugin
  skills/
    awase/
      SKILL.md            instrucciones para el agente
  profile.schema.json     estructura del perfil personal
  README.md               este fichero
```

El perfil del dev **no** está en el repo. Se guarda localmente en `~/.awase/profile.json`.
