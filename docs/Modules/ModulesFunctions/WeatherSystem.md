# `WeatherSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Tipos de Clima (`WeatherType`)](#tipos-de-clima-weathertype)
3. [Gestor de Clima (`WeatherManager`)](#gestor-de-clima-weathermanager)
4. [Impacto en el Combate](#impacto-en-el-combate)
5. [Efectos de Desastre](#efectos-de-desastre)
6. [Integración con Mods](#integración-con-mods)

---

## Descripción General

`WeatherSystem.py` introduce un mundo dinámico donde las condiciones climáticas afectan tanto el rendimiento del jugador en combate como la efectividad de los gremios. El clima cambia automáticamente cada 10 minutos de juego real.

---

## Tipos de Clima (`WeatherType`)

Cada clima se define por su probabilidad de aparición y sus multiplicadores de estadísticas. Algunos ejemplos incluidos son:

- **Soleado (☀️)**: Aumenta el ataque en un 5%.
- **Tormenta Fuerte (⛈️)**: Reduce drásticamente la Agilidad (50%) y el Contraataque (25%).
- **Niebla Densa (🌫️)**: Reduce el ataque pero aumenta la Evasión significativamente (30%).
- **Terremoto (🌋)**: Clima raro con penalizaciones severas y riesgo de destrucción de edificios.

---

## Gestor de Clima (`WeatherManager`)

Es un **Singleton** que controla la rotación de climas.
- **`check_and_update_weather()`**: Verifica si el intervalo de 10 minutos ha expirado y lanza un dado ponderado para elegir el próximo clima.
- **`_roll_new_weather()`**: Selecciona un clima basado en las probabilidades relativas de cada `WeatherType`.

---

## Impacto en el Combate

La función `apply_weather_combat_mods(base_val, mod_key)` se inyecta en los métodos de ataque y defensa de `cPlayer` y `cEnemy`. 
- **Modificadores soportados**: `attack`, `defense`, `magic_def`, `agi`, `counter`, `evasion`.
- Estos cambios se reflejan visualmente en el desglose detallado de daño en el `MainGame.py`.

---

## Efectos de Desastre

Ciertos climas extremos (Terremoto, Incendio) tienen un `disaster_effect`. 
- Poseen una probabilidad por tick de activar un evento catastrófico.
- En la simulación de Gremios, esto puede resultar en la **destrucción aleatoria de un edificio**, obligando al gremio (o al jugador) a reconstruirlo.

---

## Integración con Mods

El `WeatherManager` permite registrar climas personalizados a través de la API:
```python
api.register_weather({
    "id": "lluvia_acido",
    "name": "Lluvia Ácida",
    "icon": "🧪",
    "player_enemy_mods": {"defense": 0.5},
    "probability": 5
})
```

---

## Notas Técnicas

- El sistema utiliza `time.time()` para medir el intervalo de cambio, lo que significa que el clima persiste entre cargas de menús pero no entre sesiones (se reinicia al cargar el juego).
- Las probabilidades de todos los climas no necesitan sumar 100; el sistema normaliza los pesos automáticamente usando `random.choices`.
