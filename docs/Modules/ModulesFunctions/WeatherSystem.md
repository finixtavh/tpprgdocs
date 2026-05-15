# `WeatherSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Catálogo de Climas Base](#catalogo)

---

## 1. Descripción General
El `WeatherSystem.py` simula dinámicamente el clima mundial en TPPRPG. Los cambios de clima ocurren en intervalos de tiempo real y afectan matemáticamente tanto a los combates 1v1 (Jugador contra Enemigo) como a las guerras macro a nivel geopolítico (`GuildSystem`), pudiendo incluso desatar Desastres Naturales que destruyen edificios.

---

## 2. Dependencias e Inyecciones
- **Bibliotecas Standard**: `random`, `time` (para el cálculo de intervalos de cambio).
- No depende de JSONs externos para su configuración base; el catálogo de climas está programado a fuego en la clase (`hardcoded`), aunque expone métodos para que los *Mods* registren climas nuevos dinámicamente.

---

## 3. Constantes y Variables Globales
- **No se exponen variables globales directas**, pero la clase `WeatherManager` funciona como un **Singleton** a través de `_instance`, por lo que todos los módulos que consulten el clima verán exactamente el mismo estado.

---

## 4. Clases y Estructuras de Datos

### `WeatherType`
Objeto de solo lectura que representa un tipo de clima y sus reglas matemáticas.

#### Atributos de Instancia
- `self.id` (`str`): Identificador único (ej: `"soleado"`).
- `self.name` (`str`), `self.icon` (`str`), `self.description` (`str`): Metadatos para la Interfaz de Usuario.
- `self.combat_mods` (`Dict[str, float]`): Diccionario que define multiplicadores para los combates individuales. Contiene llaves base (`attack`, `defense`, `magic_def`, `agi`, `counter`, `evasion`) iniciadas en `1.0` y sobrescritas por los valores pasados al constructor.
- `self.guild_mods` (`Dict[str, float]`): Diccionario para los combates macro de clanes. Llaves base (`attack`, `defense`).
- `self.disaster_effect` (`Dict[str, Any]`): Información del efecto de destrucción. Formato: `{"chance": 0.15, "type": "building_destruction", "message": "..."}`. Por defecto es `{}`.
- `self.probability` (`int`): Peso estadístico para la tirada aleatoria (A mayor peso, más frecuente).

### `WeatherManager`
Motor Singleton de transición de climas.

#### Atributos de Instancia
- `self.weathers` (`Dict[str, WeatherType]`): Diccionario de todos los climas cargados.
- `self.current_weather` (`WeatherType`): Puntero al clima activo en este exacto momento.
- `self.last_change_time` (`float`): Marca de tiempo (`time.time()`) del último cambio.
- `self.change_interval` (`int`): Segundos que deben pasar antes de cambiar el clima (por defecto `600` = 10 Minutos).

#### Métodos Principales
- `__new__(cls)`: Implementa el patrón Singleton.
- `_init(self)`: Constructor privado. Inicializa el catálogo base de climas (viento, soleado, tormenta, niebla, terremoto, etc).
- `check_and_update_weather(self) -> bool`: Comprueba si `time.time() - last_change_time > change_interval`. Si es verdadero, tira los dados llamando a `_roll_new_weather()`.
- `_roll_new_weather(self)`: Usa `random.choices` tomando como pesos (`weights`) las probabilidades de cada clima.
- `get_current_weather(self) -> WeatherType`: Llama a `check_and_update_weather` (forzando actualización perezosa) y luego retorna `current_weather`.
- `register_weather(self, weather_data: Dict[str, Any])`: Permite a sistemas externos inyectar climas nuevos pasándole un diccionario con toda la información necesaria.

---

## 5. Funciones del Módulo (API)

- `get_weather_manager() -> WeatherManager`
  - **Propósito**: Retorna el Singleton del sistema climático.

- `apply_weather_combat_mods(base_val: float, mod_key: str) -> float`
  - **Propósito**: Función utilitaria rápida para el motor de combate.
  - **Parámetros**: `base_val` (el stat del jugador/enemigo, ej. `150` de Daño), `mod_key` (el nombre del stat a buscar, ej. `"attack"`).
  - **Retornos**: El valor final multiplicado. (Ej. si el daño es 150 y el clima soleado da `1.05`, retorna `157.5`).

---

## 6. Catálogo de Climas Base

| ID | Nombre | Efecto Destacado Combate 1v1 | Efecto Macro (Guilds) |
|---|---|---|---|
| `viento_suave` | Viento Suave 🍃 | Ninguno (100% todo) | Ninguno |
| `soleado` | Soleado ☀️ | Ataque +5% | Ataque +5% |
| `extra_caluroso` | Extra Caluroso 🔥 | Ataque -10%, Def. Mágica -20% | Ataque -10%, Def. -10% |
| `tormenta` | Tormenta Fuerte ⛈️ | Agilidad -50%, Contraataque -25% | Ataque -20%, Def. +10% |
| `niebla` | Niebla Densa 🌫️ | Evasión +30%, Ataque -15% | Ataque -30%, Def. +20% |
| `nevada` | Nevada Intensa ❄️ | Agilidad -20%, Defensa -10% | Ataque -20%, Def. +5% |
| `terremoto` | Terremoto 🌋 | Agilidad -70%, Evasión -50% | Atq -50%, Def -50%, **Posible Destrucción** |
| `incendio` | Incendio Forestal 🔥 | Ataque +20%, Def. Mágica -50% | Atq +10%, Def -30%, **Posible Destrucción** |
