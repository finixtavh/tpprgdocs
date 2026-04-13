# `SaveSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Constantes](#constantes)
3. [Funciones de Serialización](#funciones-de-serialización)
4. [Función `build_save_data`](#función-build_save_data)
5. [Funciones de Guardado y Carga](#funciones-de-guardado-y-carga)
6. [Funciones de Restauración Internas](#funciones-de-restauración-internas)
7. [Función `get_slot_info`](#función-get_slot_info)
8. [Función `save_load_menu`](#función-save_load_menu)
9. [Ejemplos de Uso](#ejemplos-de-uso)
10. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`SaveSystem.py` implementa el sistema completo de **guardado y carga de partidas** con soporte para **3 slots** independientes. Persiste el estado completo del juego: estadísticas del jugador, equipamiento, inventario, compañeros, árbol de habilidades, masteries y estado de la base.

Los archivos de guardado se almacenan en `Data/Saves/Slot1.json`, `Slot2.json` y `Slot3.json`.

---

## Constantes

```python
_SAVE_DIR = os.path.join("Data", "Saves")

_SLOT_PATHS = {
    1: os.path.join(_SAVE_DIR, "Slot1.json"),
    2: os.path.join(_SAVE_DIR, "Slot2.json"),
    3: os.path.join(_SAVE_DIR, "Slot3.json"),
}
```

---

## Funciones de Serialización

### `_serialize_item(item) -> Optional[dict]`

Convierte un objeto de item a un diccionario serializable. Extrae `ID`/`STATIC_ID`, nombre, categoría (`item_category` o `objectcheck`) y la lista de encantamientos aplicados.

Retorna `None` si el item es `None`.

**Formato de salida:**

```json
{
  "id": 3,
  "name": "Espada Grande",
  "category": "Weapon",
  "enchantments": [
    { "id": 1, "level": 2 }
  ]
}
```

---

### `_serialize_companion(comp) -> dict`

Serializa un compañero: `companion_id`, nombre, HP/MP actuales y máximos, si está vivo, nivel, arma y armadura equipadas.

---

### `_serialize_inventory(inventory) -> list`

Serializa todos los slots del inventario como lista de `{"item": {...}, "quantity": N}`.

---

### `_serialize_mastery(player) -> dict`

Serializa `player.mastery_manager` usando `MasteryManager.to_dict()`. Retorna `{}` si no existe el manager.

---

### `_serialize_base(player) -> dict`

Serializa `player.base_manager` usando `BaseManager.to_dict()`. Retorna `{}` si no existe.

---

## Función `build_save_data`

```python
build_save_data(player, inventory, chosen_area, active_enemies_ids=None) -> dict
```

Construye el diccionario completo de guardado.

**Estructura del save:**

```json
{
  "version": "1.0",
  "timestamp": "2025-01-15 20:30:00",
  "player": {
    "name": "...", "Level": 5, "lvlPoints": 2,
    "Health": 95, "Health_max": 140,
    "Mana": 80, "Mana_max": 110,
    "EXP": 350, "EXP_M": 500, "Gold": 1200,
    "DEF": [5, 0, 2, 0, 0],
    "AGI": 10, "DefensePercentage": [0, 0, 0, 0, 0],
    "CounterAttack": 5, "Evasion": 8,
    "ManaRegen": 2.5, "RAC": 0, "Control": 0, "Barrier": 0
  },
  "equipment": {
    "weapon": {"id": 3, "name": "Espada Grande", ...},
    "helmet": null, "armor": {...}, "boots": null,
    "ring": null, "ring2": null, "ring3": null,
    "tool": null, "SecHand": null
  },
  "inventory": [...],
  "companions": [...],
  "companion_bank": [...],
  "skills_purchased": {"skill_node_1": 1, "skill_node_2": 2},
  "mastery": {...},
  "base": {...},
  "game_state": { "chosen_area": "Forest" }
}
```

**Slots de equipamiento serializados:** `weapon`, `helmet`, `armor`, `boots`, `ring`, `ring2`, `ring3`, `tool`, `SecHand`.

---

## Funciones de Guardado y Carga

### `save_to_slot(slot: int, player, inventory, chosen_area) -> bool`

Guarda la partida en el slot indicado (1, 2 o 3). Crea el directorio `Data/Saves/` si no existe. Retorna `True` si tuvo éxito.

```python
exito = save_to_slot(1, player, vInventory, chosen_area)
```

---

### `load_from_slot(slot: int, player, inventory, item_manager) -> Optional[dict]`

Carga la partida del slot indicado. Restaura el estado completo del jugador. Retorna el dict de `game_state` (con `chosen_area`, etc.) o `None` si falla.

```python
game_state = load_from_slot(2, player, vInventory, item_manager)
if game_state:
    chosen_area = game_state.get("chosen_area")
```

---

## Funciones de Restauración Internas

### `_restore_player(player, pdata: dict)`

Restaura los stats básicos del jugador: nombre, nivel, puntos, HP, MP, EXP, oro, AGI, CounterAttack, Evasion, ManaRegen, RAC, Control, Barrier, DEF y DefensePercentage.

---

### `_restore_equipment(player, equip_data: dict, item_manager)`

Restaura el equipamiento por slot. Para cada slot, obtiene el `id` del item serializado y llama a `item_manager.create_item_object()` para reconstruir el objeto. Si el slot estaba vacío, asigna `None`.

---

### `_restore_inventory(inventory, inv_data: list, item_manager)`

Limpia el inventario actual y lo reconstruye desde los datos serializados. Cada entry llama a `item_manager.create_item_object()` y añade el item con la cantidad guardada.

---

### `_restore_companions(player, companions_data, bank_data, item_manager)`

Restaura los compañeros activos y el banco de compañeros. Usa `load_companion_by_id()` de `CompanionSystem` para reconstruir cada compañero por su `companion_id` y luego restaura su estado (HP, MP, nivel, etc.).

---

### `_restore_skills(player, skills_data: dict)`

Restaura el árbol de habilidades como diccionario `{node_id: level}`.

---

### `_restore_mastery(player, mastery_data: dict)`

Restaura el `MasteryManager` desde el diccionario usando `MasteryManager.from_dict()`.

---

### `_restore_base(player, base_data: dict)`

Restaura el `BaseManager` desde el diccionario usando `BaseManager.load_from_dict()`. Si hay error, lo imprime sin lanzar excepción.

---

## Función `get_slot_info`

```python
get_slot_info(slot: int) -> dict
```

Retorna metadatos de un slot de guardado sin cargar el estado completo. Útil para mostrar información en el menú de carga.

**Retorno:**

```python
{
    "exists": True,
    "player_name": "Héroe",
    "level": 5,
    "timestamp": "2025-01-15 20:30:00",
    "area": "Forest"
}
# O si no existe:
{"exists": False}
```

---

## Función `save_load_menu`

```python
save_load_menu(player, inventory, chosen_area, item_manager, console, menu_fn, pause_fn) -> Optional[dict]
```

Menú completo de guardar/cargar con selección de los 3 slots.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `player` | `cPlayer` | Objeto del jugador. |
| `inventory` | `cInventory` | Inventario del jugador. |
| `chosen_area` | `str \| None` | Zona actual. |
| `item_manager` | `ItemManager` | Gestor de items para reconstruir objetos al cargar. |
| `console` | `Console` | Instancia de rich Console para mostrar mensajes. |
| `menu_fn` | `Callable` | Función de menú interactivo (normalmente `interactive_menu_select`). |
| `pause_fn` | `Callable` | Función de espera (normalmente `wait_for_enter`). |

**Flujo:**

1. Muestra menú de acción: `Guardar`, `Cargar` o `Cancelar`.
2. Muestra los 3 slots con información (nombre, nivel, zona, timestamp) o `Vacío`.
3. Al guardar: llama a `save_to_slot()`.
4. Al cargar: confirma con submenú Sí/No, luego llama a `load_from_slot()`.

**Retorno:** Dict de `game_state` si se cargó exitosamente, `None` si se guardó o canceló.

---

## Ejemplos de Uso

### Guardar en slot 1

```python
from Modules.SaveSystem import save_to_slot

if save_to_slot(1, player, vInventory, chosen_area):
    print("Guardado con éxito")
```

### Cargar desde slot 2

```python
from Modules.SaveSystem import load_from_slot

game_state = load_from_slot(2, player, vInventory, item_manager)
if game_state:
    chosen_area = game_state.get("chosen_area")
    enemy = None  # Resetear enemigo
```

### Mostrar info de slots sin cargar

```python
from Modules.SaveSystem import get_slot_info

for slot_num in [1, 2, 3]:
    info = get_slot_info(slot_num)
    if info["exists"]:
        print(f"Slot {slot_num}: {info['player_name']} Nv.{info['level']} - {info['area']}")
    else:
        print(f"Slot {slot_num}: Vacío")
```

### Usar el menú completo (desde MainGame)

```python
from Modules.SaveSystem import save_load_menu

game_state = save_load_menu(
    player, vInventory, chosen_area, item_manager,
    console, interactive_menu_select, wait_for_enter
)
if game_state is not None:
    chosen_area = game_state.get("chosen_area", chosen_area)
    enemy = None
```

---

## Notas y Referencias

- Los slots de guardado son independientes entre sí: guardar en el slot 2 no afecta al 1 ni al 3.
- El campo `version` es `"1.0"`. Versiones futuras del juego pueden usar este campo para migrar saves antiguos.
- Si un item del inventario no puede reconstruirse por ID (ej: fue eliminado del JSON base), se omite silenciosamente durante la carga.
- Los compañeros se reconstruyen por `companion_id` desde `DataCompanions.json`. Si el ID no existe, el compañero se omite.
- **`json.dump` con `ensure_ascii=False`:** Permite guardar texto en español sin escapar caracteres. [https://docs.python.org/3/library/json.html](https://docs.python.org/3/library/json.html)
- **`os.makedirs(exist_ok=True)`:** Crea el directorio `Data/Saves/` de forma segura. [https://docs.python.org/3/library/os.html#os.makedirs](https://docs.python.org/3/library/os.html#os.makedirs)
