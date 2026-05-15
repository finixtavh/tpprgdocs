# `BaseSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Lógica de Requisitos Flexibles (OR)](#lógica-de-requisitos)

---

## 1. Descripción General
`BaseSystem.py` gestiona la base personal del jugador. Permite construir y mejorar cinco tipos de edificios utilizando recursos del inventario y oro. Los edificios brindan mejoras pasivas globales (como expansiones de inventario o multiplicadores de oro/EXP) y algunos ofrecen "Minijuegos" (Entrenamiento y Resistencia) que recompensan al jugador con EXP de Maestría (`MasterySystem`).

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere `./Data/DataBuildings.json` para cargar las especificaciones de los edificios y los costos de cada nivel.
- **Inyecciones Condicionales**: 
  - `ModulesUtils.Loader` (lectura de JSON).
  - `ModulesUtils.input_utils` y `sshkeyboard` (para los listeners de teclado en los minijuegos de timing).
  - `ItemManager` (inyección lazy para construir el caché de Materiales flexibles).
  - `MasterySystem` (inyección lazy para recompensar las ramas de armadura o arma tras ganar un minijuego).

---

## 3. Constantes y Variables Globales
- `_BUILDINGS_JSON` (`str`): Ruta estática `"Data/DataBuildings.json"`.
- `_BASE_SAVE_KEY` (`str`): Llave de diccionario (`"player_base"`) utilizada en la serialización.
- `_KNOWN_MATERIAL_TYPES` (`set`): Conjunto de tipos de recolección base (`{"Wood", "Rock", "Ore", "Herbs", "Leather", "Crystal", "Fiber", "Bone", "Essence", "Gem"}`).
- `_mat_type_cache` (`Dict[str, List[str]]`): Caché interna que mapea tipos abstractos (`"Wood"`) a nombres concretos de ítems (`["Madera de Roble", "Madera de Pino"]`).

---

## 4. Clases y Estructuras de Datos

### `Building`
Representa el estado actual de un edificio.

#### Atributos de Instancia
- `self.building_id` (`str`): Clave única de `DataBuildings.json`.
- `self.name` (`str`), `self.icon` (`str`), `self.description` (`str`): Textos descriptivos.
- `self.max_level` (`int`): Nivel máximo (por defecto 10).
- `self.passive_effect` (`str`): Tipo de pasiva (ej. `"+ Inventory Slots"`).
- `self.passive_value_per_level` (`float`): Cuánto aumenta el bono por nivel.
- `self.has_minigame` (`bool`): Si posee minijuego de entrenamiento.
- `self.minigame_type` (`str`): `"timing"` o `"endurance"`.
- `self.minigame_desc` (`str`): Reglas del minijuego.
- `self.level_data` (`Dict`): Información cruda de requisitos por nivel.
- `self.current_level` (`int`): Nivel actual (Mutable, `0` = No construido).

#### Propiedades Computadas
- `is_built(self) -> bool`: `True` si `current_level > 0`.
- `is_max(self) -> bool`: `True` si `current_level >= max_level`.
- `passive_total(self) -> float`: Multiplica nivel actual por `passive_value_per_level`.

#### Métodos Principales
- `next_level_data(self) -> Optional[dict]`: Obtiene el bloque JSON de los requerimientos para el nivel actual + 1. Retorna `None` si es max.
- `current_level_mastery_exp(self) -> float`: Cantidad de EXP base otorgada al pasar el minijuego de este nivel.
- `to_dict(self) -> dict` / `load_state(self, state: dict)`: Serialización para `SaveSystem.py`.

### `BaseManager`
Gestor central instanciado por cada jugador (almacenado en `player.base_manager`).

#### Atributos de Instancia
- `self.buildings` (`Dict[str, Building]`): Colección de edificios del jugador.

#### Métodos Principales
- `__init__(self)`: Instancia y llama a `_load_definitions()`.
- `_load_definitions(self)`: Parsea `DataBuildings.json` y crea las clases `Building`.
- `get_inventory_bonus_slots(self) -> int`: Suma total de inventario por el nivel del `storage` (Almacén).
- `get_exp_bonus_multiplier(self) -> float`: Multiplicador para el sistema de combate (ej. `1.10` para +10%) provisto por el `experience_hall` (Salón).
- `get_gold_bonus_multiplier(self) -> float`: Multiplicador comercial provisto por el `gold_vault` (Tesorería).
- `can_build_or_upgrade(self, building_id: str, player, inventory) -> tuple`: Valida si el jugador cumple los requisitos de Oro y Materiales (ver [Lógica de Requisitos]). Retorna `(Booleano, Razón)`.
- `build_or_upgrade(self, building_id: str, player, inventory) -> bool`: Llama a `can_build_or_upgrade`. Si es exitoso, cobra los costos, elimina recursos de inventario y muta `current_level`. Retorna booleano de éxito.

---

## 5. Funciones del Módulo (API)

### Funciones de UI y Menús
- `open_base_menu(player, inventory, console, menu_fn, pause_fn)`
  - **Propósito**: Interfaz principal (`rich.table`). Muestra estadísticas actuales (bonos pasivos) y el selector de edificios a gestionar.
- `_building_submenu(...)`
  - **Propósito**: Controlador del edificio elegido. Permite pagar la mejora, acceder al minijuego, o ver la tabla completa de costos de construcción.
- `_show_building_details(...)`
  - **Propósito**: Imprime la tabla con todos los niveles desde 1 hasta el Max, mostrando costos de oro y requerimientos dinámicos en texto legible.

### Funciones de Minijuegos
- `_run_training(building, player, console, pause_fn)`
  - **Propósito**: Controlador de minijuegos. Encamina hacia el timing o endurance y posteriormente inyecta la `Mastery_EXP` ganada a las ramas correspondientes de armaduras o armas.
- `minigame_timing(console, building) -> bool`
  - **Propósito**: Lanza un subproceso con una barra ASCII (`●`) que oscila sobre zonas de (`★`). Utiliza `sshkeyboard` para atrapar el momento en que se presiona `Enter`. Retorna victoria o derrota.
- `minigame_endurance(console, building, player) -> bool`
  - **Propósito**: Lanza 3 oleadas virtuales de daño basándose en un 10% del HP_Máx del jugador, mitigadas por las Defensas reales del jugador (`player.DEF`). Si la armadura es pobre y el jugador casi muere en la simulación, aborta y retorna derrota.

### Helpers Privados
- `_get_base_manager(player) -> BaseManager`
  - **Propósito**: Factoría que asocia e inicializa el gestor como variable del jugador en runtime (`player.base_manager`).

---

## 6. Lógica de Requisitos Flexibles (OR)
Los requerimientos en `DataBuildings.json` admiten polimorfismo. El sistema sabe diferenciar:
1. **Material Exacto**: `"Madera de Roble": 5` (Exige un objeto llamado exactamente así).
2. **Tipo de Material (OR)**: `"Wood": 5`. Lee el caché (`_build_mat_type_cache`) que consulta al `ItemManager` e identifica **todos** los objetos tipo `"Wood"`. Luego permite que el costo se pague sumando recursos mezclados (ej. `3x Madera de Roble + 2x Madera de Pino`).

Esto es operado internamente por:
- `_is_type_requirement(req_key)`
- `_names_for_type(mat_type)`
- `_count_material(inventory, req_key)`
- `_consume_material(inventory, req_key, amount)`
