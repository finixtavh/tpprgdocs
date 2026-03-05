# `BuffManager.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Variables Globales](#variables-globales)
3. [Clase `Buff`](#clase-buff)
4. [Clase `BuffManager`](#clase-buffmanager)
5. [Funciones Helper](#funciones-helper)
6. [Ejemplos de Uso](#ejemplos-de-uso)
7. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`BuffManager.py` gestiona el sistema completo de **buffs** (efectos positivos) y **debuffs** (efectos negativos) del juego. Implementa un sistema de *diminishing returns* (rendimientos decrecientes) para el apilamiento de efectos: cuantos más buffs del mismo tipo se apliquen sobre el mismo stat, menos efectivos son cada uno adicional.

Los datos de buffs y debuffs se cargan desde `./Data/DataBuffs.json`.

---

## Variables Globales

| Variable | Tipo | Descripción |
|---|---|---|
| `_buff_manager_instance` | `Optional[BuffManager]` | Instancia global Singleton del `BuffManager`. Se inicializa la primera vez que se llama a `get_buff_manager()`. |

---

## Clase `Buff`

Representa **un buff o debuff individual** aplicado a una entidad (jugador o enemigo).

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `buff_id` | `str` | Identificador único del buff (ej: `"1_buff"`, `"5_debuff"`). |
| `name` | `str` | Nombre legible del buff. |
| `description` | `str` | Descripción del efecto. |
| `duration_base` | `int` | Duración original en turnos (del JSON). |
| `duration_current` | `int` | Duración restante actual. Si es permanente, vale `9999`. |
| `type` | `str` | Tipo de cálculo: `"flat"` (valor plano) u otros. |
| `effects` | `dict` | Diccionario de efectos: `{nombre_stat: valor}`. |
| `icon` | `str` | Emoji/icono visual del buff (ej: `"✨"`). |
| `source` | `str` | Origen del buff: `"skill"`, `"item"`, `"potion"`. |
| `permanent` | `bool` | Si es `True`, el buff dura mientras el item esté equipado. |
| `buff_after_unequip` | `int` | Turnos extra de duración tras desequipar el item fuente. |
| `is_debuff` | `bool` | `True` si el ID contiene `"_debuff"`. |
| `applied` | `bool` | Tracking interno: indica si el buff ya fue aplicado. |
| `stack_penalty` | `float` | Factor de penalización por stacking (1.0 = 100%). |

### Métodos

#### `__init__(buff_id, data, source, permanent, buff_after_unequip)`

Constructor. Inicializa todos los atributos desde los datos del JSON.

#### `tick() -> bool`

Reduce la duración del buff en 1 turno. Retorna `True` si el buff sigue activo, `False` si expiró. Los buffs permanentes siempre retornan `True`.

#### `get_display_name() -> str`

Retorna el nombre con icono, turnos restantes y penalización de stacking para mostrar en pantalla.

```
✨ Fuerza Aumentada (3T) [75%]
```

#### `copy() -> Buff`

Crea y retorna una copia independiente del objeto `Buff`.

---

## Clase `BuffManager`

Gestor central que carga los datos de buffs desde JSON y aplica la lógica de *diminishing returns*.

### Constante de clase

```python
STACK_PENALTIES = {
    1: 1.00,  # Primer buff del mismo tipo: 100%
    2: 0.75,  # Segundo:  75%
    3: 0.35,  # Tercero:  35%
    4: 0.09,  # Cuarto:    9%
    5: 0.03,  # Quinto+:   3%
}
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `buffs_json_path` | `str` | Ruta al archivo JSON de datos. |
| `buffs_data` | `dict` | Diccionario con todos los buffs cargados (`BUFFS`). |
| `debuffs_data` | `dict` | Diccionario con todos los debuffs cargados (`DEBUFFS`). |

### Métodos

#### `load_buffs() -> bool`

Carga el archivo JSON especificado en `buffs_json_path`. Separa los datos en `buffs_data` y `debuffs_data`. Retorna `True` en caso de éxito.

#### `create_buff(buff_id, source, permanent, buff_after_unequip) -> Optional[Buff]`

Fábrica de buffs. Busca el buff en `buffs_data` o `debuffs_data` según el `buff_id` y retorna una instancia de `Buff`. Retorna `None` si no existe.

```python
manager = BuffManager()
buff = manager.create_buff("1_buff", source="skill")
```

#### `apply_buffs_to_entity(entity, active_buffs) -> Dict[str, float]`

Aplica todos los buffs de la lista al jugador/enemigo. Internamente:

1. Agrupa buffs por stat afectado.
2. Aplica las penalizaciones de stacking.
3. Acumula los modificadores totales.

Retorna un diccionario con el total de modificadores aplicados, por ejemplo: `{"Health_max": 50, "AGI": 10}`.

#### `tick_buffs(active_buffs) -> List[Buff]`

Llama a `tick()` en cada buff de la lista. Retorna la lista de buffs que han expirado (para que puedan ser eliminados).

#### `remove_buffs_from_entity(entity, buffs_to_remove) -> Dict[str, float]`

Revierte los efectos de una lista de buffs. Retorna un diccionario con los modificadores que se removieron.

#### `list_all_buffs(detailed=False)`

Imprime en consola todos los buffs y debuffs disponibles. Si `detailed=True`, muestra descripción y efectos completos.

#### `_group_buffs_by_stat(buffs) -> Dict[str, List[Buff]]`

*(Método interno)* Agrupa buffs por el stat que modifican. Usado para calcular penalizaciones.

#### `_apply_stacking_penalties(effects_by_stat, all_buffs)`

*(Método interno)* Aplica la penalización de `STACK_PENALTIES` a cada buff según cuántos otros buffs afectan al mismo stat. El buff de mayor valor recibe 100%, el siguiente 75%, etc.

---

## Funciones Helper

### `get_buff_manager(buffs_path) -> BuffManager`

Patrón **Singleton**. Retorna la instancia global del `BuffManager`. Si no existe, la crea.

```python
manager = get_buff_manager()
```

### `apply_buff_to_player(player, buff_id, source, permanent, buff_after_unequip)`

Función de conveniencia para aplicar un buff directamente a un jugador. Inicializa `player.active_buffs` si no existe.

### `remove_buff_from_player(player, buff_id) -> bool`

Busca y elimina el primer buff con el `buff_id` dado de la lista `player.active_buffs`.

### `process_turn_buffs(player)`

Debe llamarse al final de cada turno. Ejecuta `tick_buffs` sobre todos los buffs activos y elimina los expirados.

---

## Ejemplos de Uso

### Aplicar un buff a un jugador

```python
from Modules.BuffManager import apply_buff_to_player, process_turn_buffs

# Aplicar buff temporal de 3 turnos
apply_buff_to_player(player, "1_buff", source="skill")

# Al final del turno, reducir duración
process_turn_buffs(player)
```

### Aplicar buff permanente de equipamiento

```python
from Modules.BuffManager import apply_buff_to_player

# El buff dura mientras el item esté equipado
apply_buff_to_player(player, "2_buff", source="item", permanent=True)
```

### Crear y usar el BuffManager directamente

```python
from Modules.BuffManager import get_buff_manager

manager = get_buff_manager("./Data/DataBuffs.json")

# Crear instancia de buff
buff = manager.create_buff("1_buff", source="skill", permanent=False)
print(buff.get_display_name())  # ✨ Fuerza Aumentada (3T)

# Listar todos los buffs disponibles
manager.list_all_buffs(detailed=True)
```

### Estructura esperada en `DataBuffs.json`

```json
{
  "BUFFS": {
    "1_buff": {
      "Name": "Fuerza Aumentada",
      "Description": "Aumenta el daño físico",
      "Duration": 3,
      "Type": "flat",
      "Effects": { "DmgBoost": 10 },
      "Icon": "⚔️"
    }
  },
  "DEBUFFS": {
    "1_debuff": {
      "Name": "Veneno",
      "Description": "Daño por turno",
      "Duration": 5,
      "Type": "flat",
      "Effects": { "Health": -5 },
      "Icon": "🐍"
    }
  }
}
```

---

## Notas y Referencias

- El sistema de *diminishing returns* es un patrón común en RPGs para evitar que acumular muchos buffs del mismo tipo sea demasiado poderoso.
- **Patrón Singleton en Python:** [https://refactoring.guru/design-patterns/singleton/python/example](https://refactoring.guru/design-patterns/singleton/python/example)
- **`deepcopy` de la librería `copy`:** Crea copias independientes de objetos complejos. [https://docs.python.org/3/library/copy.html](https://docs.python.org/3/library/copy.html)
- **`typing` module:** Tipos como `Optional`, `List`, `Dict` son de este módulo. [https://docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html)
