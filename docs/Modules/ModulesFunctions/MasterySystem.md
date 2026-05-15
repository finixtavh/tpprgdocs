# `MasterySystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Ganancia de Experiencia](#flujo-de-ganancia)

---

## 1. Descripción General
`MasterySystem.py` maneja la progresión pasiva ("Mastery" o Maestría) de los jugadores con los distintos tipos de armas y armaduras. Mientras el jugador inflige daño o recibe daño en combate, este sistema recompensa la constancia aumentando niveles específicos por categoría de equipo (Ej: Espadas, Arcos, Cascos). A mayor nivel, el jugador recibe multiplicadores pasivos permanentes de daño o resistencia.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere lectura en tiempo de ejecución de `config.json` para obtener los modificadores globales de ganancia de EXP (`GLOBAL_WEAPON_MASTERY_MODIFIER` y `GLOBAL_ARMOR_MASTERY_MODIFIER`).
- **Inyecciones**: `math`, `typing`. Actúa de forma aislada sin dependencias circulares complejas, recibiendo las instancias de `player` únicamente cuando procesa daño.

---

## 3. Constantes y Variables Globales
- `MASTERY_MAX_LEVEL` (`int`): Nivel tope de maestría (20).
- `MASTERY_EXP_BASE` (`float`): Experiencia requerida para pasar de Nivel 1 a 2 (100.0).
- `MASTERY_EXP_GROWTH` (`float`): Curva exponencial de requerimiento de experiencia (1.5).
- `WEAPON_BONUS_PER_LEVEL` (`float`): +0.5% de Daño por cada nivel.
- `ARMOR_BONUS_PER_LEVEL` (`float`): +0.25% de Reducción de Daño por cada nivel.
- `WEAPON_TYPES` (`list`): `["Swords", "Staffs", "Hammers", "Spears", "Daggers", "Bows", "Generic"]`.
- `ARMOR_TYPES` (`list`): `["Helmet", "Armor", "Boots", "Ring"]`.

---

## 4. Clases y Estructuras de Datos

### `MasterySkill`
Clase contenedora de la progresión para una rama específica (ej: Solo Espadas).

#### Atributos de Instancia
- `self.name` (`str`): Nombre de UI (ej: "Espadas").
- `self.skill_type` (`str`): Categoria general (`"weapon"` o `"armor"`).
- `self.current_level` (`int`): Nivel actual.
- `self.exp_current` (`float`): Experiencia acumulada.
- `self.exp_to_next` (`float`): Experiencia necesaria para el siguiente nivel.

#### Métodos Principales
- `add_exp(self, amount: float) -> int`: Suma experiencia y aplica la matemática de Subida de Nivel. Devuelve un entero indicando si subió 1 o más niveles de golpe.
- **Propiedades Computadas**: `bonus_percent`, `bonus_multiplier`, `bonus_description` calculan matemáticamente cuánto afecta este nivel en combate real.

### `MasteryManager`
Clase factoría e indexadora atada al `player.mastery_manager`.

#### Atributos de Instancia
- `self.weapon_masteries` (`Dict[str, MasterySkill]`): Diccionario de habilidades ofensivas.
- `self.armor_masteries` (`Dict[str, MasterySkill]`): Diccionario de habilidades defensivas.

#### Métodos Principales
- `get_weapon_type(weapon) -> str` *(estático)*: Analiza el objeto del arma y determina a qué categoría de las 7 posibles pertenece (leyendo `item_category` o infiriendo por nombre).
- `register_damage_dealt(self, player, damage_total: float) -> Optional[str]`: Extrae el tipo del arma actual del jugador, calcula la EXP correspondiente (`daño / 35 * modificador global`) y llama a `add_exp`. Retorna texto para UI en caso de *Level Up*.
- `register_damage_received(self, player, damage_total: float) -> Optional[str]`: Itera a través del Casco, Armadura, Botas y Anillos equipados. Calcula la EXP (`daño / 35 * modificador global`) y aplica progreso a todas las partes a la vez.
- `get_weapon_bonus_multiplier(self, weapon) -> float`: Método rápido para que el sistema de Combate Principal aplique el daño extra de la maestría.

---

## 5. Funciones del Módulo (API)

- `ensure_mastery(player) -> MasteryManager`
  - **Propósito**: Función Singleton por jugador. Revisa si el jugador ya tiene inicializado `mastery_manager`. De no ser así, lo instancia y se lo asigna.
- `open_mastery_menu(player, console, pause_fn)`
  - **Propósito**: Abre una interfaz `rich.Panel` listando los niveles actuales de maestrías, ordenados por porcentaje de progreso y filtrando ramas no utilizadas.

---

## 6. Flujo de Ganancia de Experiencia
1. En **MainGame.py**, durante el ciclo de combate.
2. Si el jugador hace **150 de daño** con una Espada.
3. El sistema de combate llama a `register_damage_dealt`.
4. El sistema reconoce "Swords".
5. Extrae $150 / 35 = 4.28$ puntos de EXP.
6. Si hay modificador global de `config.json` de $2.0x$, la EXP final es $8.56$.
7. Si este aumento provoca subir a nivel 2, la función retorna un `string` avisando del Level Up que el motor de combate imprimirá en pantalla automáticamente.
