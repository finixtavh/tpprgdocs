# `ItemManager.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `ItemManager`](#clase-itemmanager)
3. [Variables Globales](#variables-globales)
4. [Funciones de Conveniencia](#funciones-de-conveniencia)
5. [Ejemplos de Uso](#ejemplos-de-uso)
6. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`ItemManager.py` es el sistema centralizado de **gestión de items** del juego. Carga todos los items desde `DataItems.json`, los indexa por ID y nombre para búsqueda rápida, y crea instancias de los objetos del juego (`cWeapon`, `cEquippableItems`, `cObject`) según el tipo de item.

También permite agregar, modificar y eliminar items del JSON en tiempo de ejecución.

---

## Clase `ItemManager`

### Constructor

```python
ItemManager(items_json_path="./Data/DataItems.json")
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `items_json_path` | `str` | Ruta al archivo JSON de items. |
| `items_data` | `Dict` | Datos completos del JSON en memoria. |
| `items_by_id` | `Dict[int, Dict]` | Índice de búsqueda por `STATIC_ID`. |
| `items_by_name` | `Dict[str, Dict]` | Índice de búsqueda por nombre (en minúsculas). |

---

### Métodos

#### `load_items() -> bool`

Carga el JSON de items. Si el archivo no existe, crea uno con items por defecto. Llama internamente a `_build_indexes()` para construir los índices de búsqueda. Retorna `True` en caso de éxito.

---

#### `_build_indexes()`

*(Interno)* Recorre `items_data` y llena `items_by_id` e `items_by_name` para permitir búsquedas O(1). Los datos del JSON deben tener categorías con el prefijo `"TYPE:"`.

**Estructura esperada en el JSON:**

```json
{
    "TYPE:Weapons": {
        "wooden_sword": {
            "STATIC_ID": 1,
            "Name": "Espada de Madera",
            "Type": "Weapon",
            "DMG": [10, 0, 0, 0, 0],
            "DMG_min": [5, 0, 0, 0],
            "Gold_Cost": 10,
            "Description": "Una espada básica"
        }
    },
    "TYPE:Armor": {
        "leather_armor": {
            "STATIC_ID": 10,
            "Name": "Armadura de Cuero",
            "Type": "Armor",
            "Defense": [5, 2, 2, 2, 1],
            "HealthBoost": 10,
            "Gold_Cost": 30
        }
    }
}
```

---

#### `get_item_by_id(item_id: int) -> Optional[Dict]`

Retorna los datos del item con el `STATIC_ID` dado, o `None` si no existe.

```python
data = manager.get_item_by_id(3)
```

---

#### `get_item_by_name(name: str) -> Optional[Dict]`

Retorna los datos del item por nombre (case-insensitive). Busca con `name.lower()`.

```python
data = manager.get_item_by_name("Espada de Madera")
```

---

#### `get_items_by_type(item_type: str) -> List[Dict]`

Retorna todos los items de una categoría. Busca la clave `"TYPE:{item_type}"` en el JSON.

```python
weapons = manager.get_items_by_type("Weapons")
```

---

#### `create_item_object(item_id, cWeapon, cEquippableItems, cObject)`

**Método central del sistema.** Crea y retorna una instancia del objeto de item correcto según su `Type`:

| `Type` en JSON | Clase instanciada | Método interno |
|---|---|---|
| `"Weapon"` | `cWeapon` | `_create_weapon()` |
| `"Armor"`, `"Helmet"`, `"Boots"`, `"Ring"` | `cEquippableItems` | `_create_equippable()` |
| `"Consumable"` | `cObject` | `_create_consumable()` |

Retorna `None` si el item no existe o el tipo es desconocido.

---

#### `_create_weapon(item_data, cWeapon)`

*(Interno)* Normaliza el array de daño al nuevo formato de 5 valores y crea una instancia de `cWeapon`. Maneja compatibilidad con formatos antiguos:

| Formato de entrada `DMG` | Conversión |
|---|---|
| Lista de 5+ valores | Sin cambio |
| Lista de 2 valores `[Físico, Mágico]` | `[F, M, 0, 0, 0]` |
| Número único | `[N, 0, 0, 0, 0]` |
| Inválido | `[0, 0, 0, 0, 0]` |

El mismo proceso aplica para `DMG_min` (4 valores).

---

#### `_create_equippable(item_data, cEquippableItems)`

*(Interno)* Normaliza el array de defensa y crea una instancia de `cEquippableItems`.

| Formato de entrada `Defense` | Conversión |
|---|---|
| Lista de 5+ valores | Sin cambio |
| Lista de 2 valores `[F, M]` | `[F, M, M, M, M]` |
| Número único | `[N, 0, 0, 0, 0]` |
| Inválido | `[0, 0, 0, 0, 0]` |

---

#### `_create_consumable(item_data, cObject)`

*(Interno)* Crea una instancia de `cObject` (poción u objeto consumible) con `HealthRestore` y `ManaRestore`.

---

#### `add_item(item_type, item_key, item_data) -> bool`

Agrega un nuevo item al JSON y reconstruye los índices. Si la categoría no existe, la crea.

---

#### `modify_item(item_id, updates) -> bool`

Actualiza campos específicos de un item existente (identificado por `STATIC_ID`) y guarda los cambios.

```python
manager.modify_item(3, {"Gold_Cost": 250, "Description": "Nueva descripción"})
```

---

#### `delete_item(item_id) -> bool`

Elimina un item del JSON por su `STATIC_ID` y guarda los cambios.

---

#### `save_items() -> bool`

Persiste el estado de `items_data` en el archivo JSON con indentación de 4 espacios.

---

#### `list_all_items(detailed=False)`

Imprime en consola todos los items disponibles. Si `detailed=True`, muestra descripción, stats de daño, defensa, bonificaciones, etc.

---

## Variables Globales

| Variable | Tipo | Descripción |
|---|---|---|
| `_item_manager_instance` | `Optional[ItemManager]` | Instancia global Singleton del `ItemManager`. |

---

## Funciones de Conveniencia

### `get_item_manager(items_path) -> ItemManager`

Patrón **Singleton**. Crea la instancia global la primera vez (llamando a `load_items()`) y la retorna en llamadas sucesivas.

```python
from Modules.ItemManager import get_item_manager
manager = get_item_manager()
```

---

### `load_item_by_id(item_id, cWeapon, cEquippableItems, cObject)`

Atajo para cargar un item por ID y obtener el objeto del juego directamente.

```python
from Modules.ItemManager import load_item_by_id
espada = load_item_by_id(3, cWeapon, cEquippableItems, cObject)
```

---

### `load_items_by_type(item_type, cWeapon, cEquippableItems, cObject) -> List`

Carga todos los items de un tipo y retorna una lista de objetos del juego instanciados.

```python
from Modules.ItemManager import load_items_by_type
todas_las_armas = load_items_by_type("Weapons", cWeapon, cEquippableItems, cObject)
```

---

### `search_item_by_name(name) -> Optional[Dict]`

Busca un item por nombre (delega a `get_item_manager().get_item_by_name(name)`).

---

## Ejemplos de Uso

### Cargar un item y añadirlo al inventario

```python
from Modules.ItemManager import load_item_by_id
from Modules.setup import cWeapon, cEquippableItems, cObject, vInventory

pocion = load_item_by_id(52, cWeapon, cEquippableItems, cObject)
if pocion:
    vInventory.addObject(pocion)
    print(f"Añadido: {pocion.name}")
```

### Agregar un item nuevo al JSON en tiempo de ejecución

```python
from Modules.ItemManager import get_item_manager

manager = get_item_manager()
manager.add_item("Weapons", "dragon_blade", {
    "STATIC_ID": 999,
    "Name": "Hoja del Dragón",
    "Type": "Weapon",
    "DMG": [80, 30, 0, 0, 20],
    "DMG_min": [50, 15, 0, 0],
    "Gold_Cost": 5000,
    "Description": "Un arma legendaria forjada con escamas de dragón"
})
```

### Modificar el costo de un item

```python
manager = get_item_manager()
manager.modify_item(1, {"Gold_Cost": 15})  # Bajar el precio de la Espada de Madera
```

---

## Notas y Referencias

- Los `STATIC_ID` deben ser **únicos** en todo el JSON. Si dos items tienen el mismo ID, el índice guardará solo uno de ellos.
- El sistema de tipos usa el campo `"Type"` del JSON (no la categoría). Asegúrate de que coincida exactamente con: `"Weapon"`, `"Armor"`, `"Helmet"`, `"Boots"`, `"Ring"`, `"Consumable"`.
- **`json` módulo de Python:** Para leer y escribir JSON. [https://docs.python.org/3/library/json.html](https://docs.python.org/3/library/json.html)
- **Patrón Singleton:** [https://refactoring.guru/design-patterns/singleton/python/example](https://refactoring.guru/design-patterns/singleton/python/example)
- **`os.makedirs(..., exist_ok=True)`:** Crea directorios de forma segura sin error si ya existen. [https://docs.python.org/3/library/os.html#os.makedirs](https://docs.python.org/3/library/os.html#os.makedirs)
