---
name: clima
description: Consulta el clima actual y el pronóstico de una ciudad (por defecto Ciudad Juárez, Chihuahua) usando wttr.in. Úsala cuando el usuario pregunte por el clima, la temperatura, si va a llover, o invoque /clima.
argument-hint: "[ciudad]"
allowed-tools: Bash(curl:*)
---

# Clima

Obtén el clima con `curl` contra [wttr.in](https://wttr.in) (gratis, sin API key) y responde **en español**.

## Ubicación

- Si el usuario indica una ciudad (`$ARGUMENTS` o en su mensaje), úsala. Reemplaza espacios por `+` y codifica acentos con URL encoding (ej. `Ciudad+Ju%C3%A1rez`).
- Si no indica ninguna, usa por defecto la ciudad donde radica el usuario: **Ciudad Juárez, Chihuahua** → `Ciudad+Ju%C3%A1rez,Chihuahua`.

## Pasos

1. Clima actual (una línea, métrico):

   ```bash
   curl -s "https://wttr.in/<ciudad>?format=%l:+%c+%C,+%t+(sensaci%C3%B3n+%f),+humedad+%h,+viento+%w,+lluvia+%p&lang=es&m"
   ```

2. Si el usuario pide pronóstico, o pregunta por mañana / los próximos días, consulta el JSON y resume hasta 3 días (máx/mín, condición, probabilidad de lluvia):

   ```bash
   curl -s "https://wttr.in/<ciudad>?format=j1&lang=es&m"
   ```

   Campos útiles: `current_condition[0]` (`temp_C`, `FeelsLikeC`, `humidity`, `lang_es[0].value`), `weather[i]` (`date`, `maxtempC`, `mintempC`, `hourly[*].chanceofrain`, `hourly[4].lang_es[0].value` ≈ mediodía), `nearest_area[0]` (`areaName`, `country`).

## Respuesta

- Breve: ubicación, condición, temperatura y sensación térmica, humedad, viento y, si aplica, pronóstico en una lista corta.
- Usa unidades métricas (°C, km/h) salvo que el usuario pida otras (para °F usa `&u` en lugar de `&m`).
- Si `curl` falla o la respuesta contiene `Unknown location`, dilo claramente y pide una ciudad más específica (ej. agregar país: `Juarez,Mexico`).
