---
name: clima-local
description: Obtiene el clima actual y el pronóstico de los próximos días para San Miguel de Tucumán, Argentina (por defecto) o una ciudad que el usuario indique. Úsala cuando pregunte por el clima, la temperatura, si va a llover, el pronóstico, o "cómo está el tiempo".
---

# Clima local

Consulta el clima sin API key usando servicios gratuitos. Responde siempre en español.

## Ubicación por defecto

**San Miguel de Tucumán, Tucumán, Argentina** (lat `-26.8241`, lon `-65.2226`).

Usa esta ubicación siempre, salvo que el usuario pida explícitamente otra ciudad. No detectes la ubicación por IP.

## Pasos

1. **Determinar la ubicación**
   - Si el usuario indicó otra ciudad, úsala.
   - Si no, usa `San+Miguel+de+Tucuman,Argentina`.

2. **Consultar el clima** (funciona igual en Bash y PowerShell):

   ```
   curl -s "https://wttr.in/San+Miguel+de+Tucuman,Argentina?format=j1&lang=es"
   ```

   - Para otra ciudad, reemplaza el nombre y los espacios por `+` (ej. `Buenos+Aires`).
   - Campos útiles del JSON:
     - `current_condition[0]`: `temp_C`, `FeelsLikeC`, `humidity`, `windspeedKmph`, `winddir16Point`, `precipMM`, `uvIndex`, `lang_es[0].value` (descripción).
     - `nearest_area[0]`: `areaName[0].value`, `country[0].value` (confirma la ubicación).
     - `weather[]`: un elemento por día con `date`, `maxtempC`, `mintempC`, `hourly[].chanceofrain`.

3. **Fallback** si `wttr.in` falla o no responde: Open-Meteo.
   - Para Tucumán usa las coordenadas por defecto directamente. Para otra ciudad, geocodifica: `curl -s "https://geocoding-api.open-meteo.com/v1/search?name=<CIUDAD>&count=1&language=es"`
   - Clima: `curl -s "https://api.open-meteo.com/v1/forecast?latitude=-26.8241&longitude=-65.2226&current=temperature_2m,apparent_temperature,relative_humidity_2m,wind_speed_10m,precipitation,weather_code&daily=temperature_2m_max,temperature_2m_min,precipitation_probability_max&timezone=auto"`

4. **Responder** de forma breve:
   - Ubicación (ciudad, país).
   - Ahora: condición, temperatura (sensación térmica), humedad, viento.
   - Próximos 3 días: mín/máx y probabilidad de lluvia.
   - Una línea práctica si aplica (llevar paraguas, abrigarse, UV alto).

## Notas

- Unidades: °C y km/h.
- No guardes ni imprimas datos innecesarios del JSON crudo.
- Si no hay red o ambos servicios fallan, dilo claramente en vez de inventar datos.
