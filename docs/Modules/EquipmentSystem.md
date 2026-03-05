# `EquipmentSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `EquipmentManager`](#clase-equipmentmanager)
3. [Función `equip_menu`](#función-equip_menu)
4. [Sistema de Defensa Multi-tipo](#sistema-de-defensa-multi-tipo)
5. [Ejemplos de Uso](#ejemplos-de-uso)
6. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`EquipmentSystem.py` gestiona de forma dinámica el **equipamiento del jugador**: equipar y desequipar items, aplicar y remover sus stats automáticamente, y mostrar el estado actual del equipo. Soporta el sistema de defensa multi-tipo con 5 valores (Físico, Térmico, Tierra, Eléctrico, Profundo).

---

## Clase `EquipmentManager`

Clase estática (todos sus métodos son `@staticmethod`) que centraliza la lógica de equipamiento.

### Constante de clase

```python
VALID_SLOTS = {
    "Weapon":  "weapon",
    "Helmet":  "helmet",
    "Armor":   "armor",
    "Boots":   "boots",
    "Ring":    ["ring", "ring2", "ring3"]  # 3 slots de anillo
}
```

Mapea el tipo de item (según `item.objectcheck`) al nombre del atributo del jugador donde se guarda el item equipado.

---

### Métodos

#### `get_item_stats(item) -> Dict`

Extrae los stats que un item proporciona al ser equipado.

**Stats extraídos:**

| Atributo del item | Key en el dict retornado | Condición |
|---|---|---|
| `Defense` | `'DEF'` | Siempre; se normaliza a lista de 5 valores. |
| `HealthBoost` | `'Health_max'` | Solo si `> 0`. |
| `DmgBoost` | `'DmgBoost'` | Solo si `> 0`. |
| `HealthSteal` | `'HealthSteal'` | Solo si `> 0`. |

> Las armas (`cWeapon`) no aplican stats automáticamente; su daño se calcula en el método `attack()`.

---

#### `apply_item_stats(player, item)`

Aplica los stats del item al jugador. Para `DEF`, suma cada valor del array al array de defensa del jugador por tipo. Imprime en consola los stats añadidos. Al finalizar, marca `item.AppliedStats = True` para evitar doble aplicación.

---

#### `remove_item_stats(player, item)`

Revierte los stats de un item (los resta del jugador). Solo actúa si `item.AppliedStats == True`. Al finalizar, marca `item.AppliedStats = False`.

---

#### `find_available_ring_slot(player) -> Optional[str]`

Busca el primer slot de anillo disponible (`ring`, `ring2`, `ring3`). Retorna el nombre del slot o `None` si los tres están ocupados.

---

#### `equip_item(player, item, inventory=None) -> bool`

Equipa un item en el slot correspondiente. Flujo:

1. Verifica que el item tenga `objectcheck` (que sea equipable).
2. Para `"Ring"`, busca un slot disponible.
3. Si el slot ya tiene un item, lo desequipa y lo devuelve al inventario.
4. Equipa el nuevo item y aplica sus stats.

Retorna `True` si se equipó correctamente.

```python
EquipmentManager.equip_item(player, mi_espada, vInventory)
```

---

#### `unequip_item(player, slot_name, inventory=None) -> Optional[item]`

Desequipa el item del slot indicado. Slots válidos: `"weapon"`, `"helmet"`, `"armor"`, `"boots"`, `"ring"`, `"ring2"`, `"ring3"`. Remueve sus stats, pone el slot en `None` y opcionalmente devuelve el item al inventario.

---

#### `show_equipment(player)`

Imprime en consola el equipamiento actual del jugador con todos sus stats desglosados por tipo de daño/defensa.

---

#### `get_total_stats(player) -> Dict`

Retorna un diccionario con los stats actuales del jugador (incluyendo los del equipamiento ya aplicado). Útil para mostrar en el menú de estadísticas.

**Stats incluidos:** `Health`, `Health_max`, `Mana`, `Mana_max`, `DEF`, `MDEF`, `AGI`, `Level`, `EXP`, `EXP_M`, `Gold`.

---

## Función `equip_menu`

```python
equip_menu(player, inventory)
```

Menú interactivo para gestionar el equipamiento desde la consola.

### Opciones del menú

| Opción | Acción |
|---|---|
| `1` | Listar items equipables del inventario y equipar el seleccionado. |
| `2` | Mostrar slots para desequipar y desequipar el seleccionado. |
| `3` | Mostrar el equipamiento actual con stats. |
| `0` | Salir del menú. |

---

## Sistema de Defensa Multi-tipo

El juego usa un array de 5 valores para representar la defensa:

```python
DEF = [Physical, Thermal, Earth, Electric, Deep]
#      índice 0    1        2       3         4
```

Los nombres de los tipos de daño para mostrar:

```python
damage_type_names = ["Físico", "Térmico", "Tierra", "Eléctrico", "Profundo"]
```

Cuando se aplica defensa de un item, se suman los valores posición por posición. Cuando se desequipa, se restan.

---

## Ejemplos de Uso

### Equipar un item directamente

```python
from Modules.EquipmentSystem import EquipmentManager

# Equipar sin devolver el item anterior al inventario
EquipmentManager.equip_item(player, armadura_de_hierro)

# Equipar devolviendo el anterior al inventario
EquipmentManager.equip_item(player, armadura_de_hierro, vInventory)
```

### Desequipar un slot

```python
# Desequipar la armadura
item = EquipmentManager.unequip_item(player, "armor", vInventory)
if item:
    print(f"{item.name} desequipado y devuelto al inventario")
```

### Ver el equipamiento en consola

```python
EquipmentManager.show_equipment(player)
```

### Extraer stats de un item

```python
stats = EquipmentManager.get_item_stats(mi_armadura)
# {'DEF': [10, 0, 5, 0, 0], 'Health_max': 50}
```

### Abrir el menú interactivo

```python
from Modules.EquipmentSystem import equip_menu
equip_menu(player, vInventory)
```

---

## Notas y Referencias

- El flag `item.AppliedStats` (booleano en `cEquippableItems`) es crítico para evitar que los stats se apliquen múltiples veces. Siempre debe ser `False` por defecto en items nuevos.
- **`getattr` y `setattr`:** Usados para acceder y modificar atributos del jugador dinámicamente. Documentación: [https://docs.python.org/3/library/functions.html#getattr](https://docs.python.org/3/library/functions.html#getattr)
- **`isinstance`:** Permite verificar el tipo de un objeto en tiempo de ejecución. [https://docs.python.org/3/library/functions.html#isinstance](https://docs.python.org/3/library/functions.html#isinstance)
- La librería `keyboard` se importa dentro de `equip_menu` para manejar la espera de confirmación. Requiere instalación: `pip install keyboard`.
