---
name: clima
description: Consulta el clima actual y el pronóstico de la ubicación local del usuario (o de una ciudad indicada). Úsala cuando pida "el clima", "qué tiempo hace", "va a llover", temperatura o pronóstico.
argument-hint: "[ciudad opcional]"
allowed-tools: Bash(curl *)
---

# Clima local

Obtiene el clima con [wttr.in](https://wttr.in) (sin API key). Responde siempre en español.

## Pasos

1. Determina la ubicación:
   - Si el usuario pasó una ciudad (`$ARGUMENTS`), úsala (reemplaza espacios por `+`).
   - Si no, deja la ubicación vacía: wttr.in la detecta por IP.
2. Ejecuta:

   ```bash
   curl -s "https://wttr.in/<ciudad>?format=j1&lang=es"
   ```

   Ejemplo sin ciudad: `curl -s "https://wttr.in/?format=j1&lang=es"`.
3. Del JSON resultante usa:
   - `nearest_area[0]` → ciudad (`areaName`), región y país.
   - `current_condition[0]` → `temp_C`, `FeelsLikeC`, `humidity`, `windspeedKmph`, `winddir16Point`, `lang_es[0].value` (descripción), `uvIndex`.
   - `weather[0..2]` → `date`, `maxtempC`, `mintempC` y `hourly[].chanceofrain` para el pronóstico de 3 días.
4. Presenta un resumen breve:

   ```
   📍 <Ciudad, País>
   Ahora: <temp>°C (sensación <x>°C) · <descripción>
   Humedad <x>% · Viento <x> km/h <dirección> · UV <x>
   Hoy / Mañana / Pasado: <min>–<max>°C, lluvia máx. <x>%
   ```

5. Si es relevante, añade un consejo corto (paraguas, protector solar, abrigo).

## Notas

- Si `curl` falla o la respuesta no es JSON válido, informa del error y sugiere indicar la ciudad explícitamente.
- La detección por IP puede ser imprecisa; menciona la ubicación detectada para que el usuario la confirme.
- Usa unidades métricas (°C, km/h).
