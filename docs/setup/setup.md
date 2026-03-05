# `setup.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Variables Globales](#variables-globales)
3. [Clase `cPlayer`](#clase-cplayer)
4. [Clase `cEnemy`](#clase-cenemy)
5. [Clase `cWeapon`](#clase-cweapon)
6. [Clase `cEquippableItems`](#clase-cequippableitems)
7. [Clase `cObject`](#clase-cobject)
8. [Clase `cInventory` e `InventorySlot`](#clase-cinventory-e-inventoryslot)
9. [Clase `cDebuff`](#clase-cdebuff)
10. [Clase `cBuffs`](#clase-cbuffs)
11. [Clase `cIDSorter`](#clase-cidsorter)
12. [Clase `cShop`](#clase-cshop)
13. [Clases de Skills](#clases-de-skills)
14. [Instancias Globales](#instancias-globales)
15. [Funciones Globales](#funciones-globales)
16. [Sistema de Daño Multi-tipo](#sistema-de-daño-multi-tipo)
17. [Ejemplos de Uso](#ejemplos-de-uso)
18. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`setup.py` es el **núcleo del juego**. Define todas las clases principales del RPG: jugador, enemigos, armas, equipamiento, inventario, buffs, debuffs y tiendas. También inicializa las instancias globales que son compartidas por todos los módulos.

---

## Variables Globales

| Variable | Tipo | Descripción |
|---|---|---|
| `player` | `None` → `cPlayer` | Referencia al jugador. Inicialmente `None`, se asigna con `setup_player()`. |
| `LObject` | `list` | Lista global que acumula todos los items instanciados (`cWeapon`, `cEquippableItems`, `cObject`). |
| `console` | `Console` | Instancia de `rich.Console` para salida formateada. |

---

## Clase `cPlayer`

Representa al personaje del jugador. Es la clase más compleja del juego.

### Constructor

```python
cPlayer(name: str, weapon: cWeapon | None)
```

### Atributos de instancia

#### Stats principales

| Atributo | Tipo | Valor inicial | Descripción |
|---|---|---|---|
| `name` | `str` | — | Nombre del personaje. |
| `Health_max` | `int` | `100` | HP máximo. |
| `Health` | `int` | `100` | HP actual. |
| `Mana_max` | `int` | `100` | Maná máximo. |
| `Mana` | `int` | `100` | Maná actual. |
| `Level` | `int` | `1` | Nivel del personaje. |

#### Stats secundarios

| Atributo | Tipo | Valor inicial | Descripción |
|---|---|---|---|
| `DEF` | `list[5]` | `[0,0,0,0,0]` | Defensa por tipo: [Físico, Térmico, Tierra, Eléctrico, Profundo]. |
| `MDEF` | `int` | `0` | Defensa mágica (pendiente de eliminación). |
| `AGI` | `int` | `0` | Agilidad: % de probabilidad de acertar ataques. |
| `DefensePercentage` | `int` | `0` | % de daño bloqueado (ignora daño de debuffs). |
| `CounterAttack` | `int` | `0` | % de contraatacar sin consumir turno. |
| `Evasion` | `int` | `0` | % de esquivar daño entrante. |
| `ManaRegen` | `float` | `2.5` | Maná regenerado por turno. |

#### Economía y progresión

| Atributo | Tipo | Valor inicial | Descripción |
|---|---|---|---|
| `Gold` | `int` | `50` | Oro actual. |
| `EXP` | `int` | `15` | Experiencia actual. |
| `EXP_M` | `int` | `100` | EXP necesaria para subir de nivel. |

#### Slots de equipamiento

| Atributo | Tipo | Descripción |
|---|---|---|
| `weapon` | `cWeapon \| None` | Arma equipada. |
| `helmet` | `cEquippableItems \| None` | Casco equipado. |
| `armor` | `cEquippableItems \| None` | Armadura equipada (inicia con una armadura básica). |
| `boots` | `cEquippableItems \| None` | Botas equipadas. |
| `ring`, `ring2`, `ring3` | `cEquippableItems \| None` | Hasta 3 anillos. |

#### Listas de estado

| Atributo | Tipo | Descripción |
|---|---|---|
| `applieddebuffs` | `list` | Debuffs activos aplicados al jugador. |
| `appliedbuffs` | `list` | Buffs activos del jugador. |
| `companion` | `list` | Lista de compañeros del jugador. |
| `items_in_PlayerClass` | `list` | Lista de todos los slots de equipamiento [weapon, helmet, armor, boots, ring, ring2, ring3]. |
| `lobjectscheck` | `list[tuple]` | Lista de tuplas `(item, tipo_esperado)` para validación de slots. |

### Métodos

#### `attack(enemy) -> float | int`

Ataca al enemigo. Si no tiene arma, hace 1-3 de daño físico. Con arma:

1. Llama a `self.weapon.use_weapon(self)` para obtener el array de daño `[5]`.
2. Resta la defensa del enemigo a cada tipo de daño.
3. Llama a `enemy.TakeDamage(final_damage)`.
4. Imprime el desglose de daño.

Retorna el daño total infligido.

---

#### `TakeDamage(damage)`

Reduce el HP del jugador. Acepta un número único (compatibilidad) o una lista de 5 valores. El HP no baja de 0.

---

#### `win(quantity, quantity2)`

Llamar cuando el jugador derrota un enemigo. Añade `quantity` EXP y `quantity2` oro. Maneja la subida de nivel automáticamente.

---

#### `PlayerObjects()`

Imprime en consola los atributos de todos los items equipados.

---

#### `Check(lInventory)`

**Método de validación principal.** Se llama al inicio de cada turno. Realiza:

1. Actualiza `items_in_PlayerClass` y `lobjectscheck` con el estado actual.
2. Llama a `Companion_Updater(self)`.
3. Clampea HP y Maná a sus máximos.
4. Si algún slot tiene un `cObject` (consumible), lo mueve al inventario.
5. Si el jugador murió (HP ≤ 0), restaura HP y aplica penalización de oro.
6. Aplica stats de items equipados que aún no han sido aplicados (`AppliedStats == False`).
7. Verifica que los items estén en el slot correcto según su `objectcheck`.
8. Gestiona la subida de nivel y muestra el mensaje formateado.

---

#### `Equip(vInventory, InventorySortervar, player)`

Menú antiguo de equipamiento (consola). Se recomienda usar `equip_menu` de `EquipmentSystem.py` en su lugar.

---

## Clase `cEnemy`

Representa un enemigo del juego.

### Constructor

```python
cEnemy(NAME, HP_M, DMG, DMG_min, MP_M, EXP_TO_PLAYER, DEF, LS, AGI,
       DBUFF_TO_PLAYER, BUFF_RECEIVED, GOLD)
```

### Atributos

| Atributo | Tipo | Descripción |
|---|---|---|
| `NAME` | `str` | Nombre del enemigo. |
| `HP_M` | `int` | HP máximo. |
| `HP` | `int` | HP actual (= HP_M al inicio). |
| `DMG` | `list[5]` | Daño máximo por tipo: [Físico, Térmico, Tierra, Eléctrico, Profundo]. |
| `DMG_min` | `list[5]` | Daño mínimo por tipo (Deep siempre es máximo). |
| `DEF` | `list[5]` | Defensa por tipo. |
| `MP_M` | `int` | Maná máximo. |
| `MP` | `int` | Maná actual. |
| `EXP_TO_PLAYER` | `int` | EXP otorgada al morir. |
| `LS` | `int` | Robo de vida. |
| `AGI` | `int` | Agilidad. |
| `DBUFF_TO_PLAYER` | `list` | Debuffs que aplica al atacar. |
| `BUFF_RECEIVED` | `list` | Buffs que recibe al inicio del combate. |
| `GOLD` | `int` | Oro soltado al morir. |

### Métodos

#### `attack(player) -> float`

Ataca al jugador. Calcula daño aleatorio entre `DMG_min[i]` y `DMG[i]` para cada tipo. Deep siempre hace daño máximo. Llama a `player.TakeDamage()`. Retorna el daño total.

#### `TakeDamage(damage)`

Reduce el HP del enemigo. Acepta número o lista de 5 valores.

---

## Clase `cWeapon`

Representa un arma equipable.

### Constructor

```python
cWeapon(ID, name, DMG, DMG_min, Gold_Cost, Description)
```

### Atributos

| Atributo | Tipo | Descripción |
|---|---|---|
| `ID` | `int` | ID único. |
| `name` | `str` | Nombre del arma. |
| `DMG` | `list[5]` | Daño máximo: [Físico, Térmico, Tierra, Eléctrico, Profundo]. |
| `DMG_min` | `list[4]` | Daño mínimo: [Físico, Térmico, Tierra, Eléctrico] (Deep siempre máximo). |
| `Gold_Cost` | `int` | Costo en oro. |
| `Description` | `str` | Descripción. |
| `AppliedStats` | `bool` | Siempre `False` para armas (los stats se calculan en cada ataque). |
| `PHYSICAL_DMG` | `int` | Alias de `DMG[0]` para compatibilidad. |
| `damage_min` | `int` | Alias de `DMG_min[0]` para compatibilidad. |
| `damage_max` | `int` | Alias de `DMG[0]` para compatibilidad. |

### Método `use_weapon(player=None) -> list[5]`

Calcula el array de daño para un ataque:

1. Para cada tipo con `DMG[i] > 0`: genera aleatorio entre `DMG_min[i]` y `DMG[i]` (excepto Deep, que siempre es máximo).
2. Suma `DmgBoost` de anillos y armadura al daño físico.

Retorna una lista de 5 valores de daño.

---

## Clase `cEquippableItems`

Representa un item equipable (armadura, casco, botas, anillo).

### Constructor

```python
cEquippableItems(ID, name, Defense, HealthBoost, HealthSteal, DmgBoost, Gold_Cost, Description, objectcheck)
```

### Atributos

| Atributo | Tipo | Descripción |
|---|---|---|
| `ID` | `int` | ID único. |
| `name` | `str` | Nombre del item. |
| `Defense` | `list[5]` | Defensa por tipo de daño. Normalizado siempre a 5 valores. |
| `HealthBoost` | `int` | Bonus de HP máximo al equipar. |
| `HealthSteal` | `int` | Robo de vida por ataque. |
| `DmgBoost` | `int` | Bonus de daño físico. |
| `Gold_Cost` | `int` | Costo en oro. |
| `Description` | `str` | Descripción. |
| `objectcheck` | `str` | Tipo del item: `"Weapon"`, `"Helmet"`, `"Armor"`, `"Boots"`, `"Ring"`. |
| `AppliedStats` | `bool` | `False` si los stats aún no se han aplicado al jugador. |

---

## Clase `cObject`

Objeto consumible (pociones, etc.).

### Constructor

```python
cObject(ID, name, HealthRestore, ManaRestore, Gold_Cost, Description)
```

| Atributo | Descripción |
|---|---|
| `HealthRestore` | HP restaurado al usar. |
| `ManaRestore` | Maná restaurado al usar. |

---

## Clase `cInventory` e `InventorySlot`

### `InventorySlot`

Representa un slot del inventario con soporte para stacking.

| Atributo | Descripción |
|---|---|
| `item` | El item en el slot. |
| `quantity` | Cantidad actual. |
| `max_stack` | Máximo de items apilables (de `item.MaxStack`, default `1`). |

**Métodos:** `can_add(amount)`, `add(amount) -> overflow`, `remove(amount) -> removido`, `is_empty()`, `get_display_name()`.

### `cInventory`

Inventario con sistema de stacking automático.

#### Métodos importantes

| Método | Descripción |
|---|---|
| `addObject(item, quantity=1, appenditem=True)` | Añade item. Si `appenditem=False`, lo remueve. |
| `remove_item(item, quantity=1) -> int` | Remueve items por objeto o por ID entero. |
| `get_item_count(item_id) -> int` | Cantidad total de un item. |
| `has_item(item_id, quantity=1) -> bool` | Verifica si hay suficiente cantidad. |
| `PrintObjects()` | Muestra el inventario en consola. |

#### Propiedades de compatibilidad

| Propiedad | Descripción |
|---|---|
| `lObjectsInventory` | Lista plana de todos los items (sin stacks). |
| `Objects` | Alias de `lObjectsInventory`. |

---

## Clase `cDebuff`

Representa un debuff aplicable al jugador.

### Efectos disponibles

| Parámetro | Efecto |
|---|---|
| `PersistentDamageDebuff` | Daño por turno (no implementado). |
| `HealDebuff` | Reduce el HP del jugador. |
| `ArmorPierce` | Reduce la defensa del jugador. |
| `AgilityDebuff`, `MPDisruption`, `EvadeDebuff`, `CounterAttackDebuff` | No implementados. |

### Método `Apply(Value, player)`

Aplica el efecto del debuff al jugador. `Value` es el nombre del atributo del debuff a aplicar.

---

## Clase `cBuffs`

Representa un buff para el jugador.

| Atributo | Descripción |
|---|---|
| `EvadeBuff` | Bonus de evasión. |
| `Heal` | Curación. |
| `MagicArmor` | Armadura mágica. |
| `MPregen` | Regeneración de maná. |
| `Strength` | Bonus de fuerza. |

---

## Clase `cIDSorter`

Utilidad para buscar items por ID en una lista.

### Método `SearchObject(ID)`

Retorna el primer objeto de la lista cuyo atributo `ID` coincida. Retorna `None` si no se encuentra.

---

## Clase `cShop`

Tienda simple (sistema antiguo, reemplazado por `ShopManager`).

### Método `buy()`

Compra el item si el jugador tiene suficiente oro. Descuenta el oro y añade el item al inventario.

---

## Clases de Skills

### `cSkill` (dataclass)

```python
@dataclass
class cSkill:
    exp_total: float = 0.0
    exp_to_next_lvl: float = 100.0
    current_level: int = 1
```

### `cFarm`

Versión base de la clase de skills en `setup.py`. El método `gather()` aquí no está implementado (usa `NotImplemented`). La versión funcional está en `GatheringMasterySystem.py`.

---

## Instancias Globales

| Instancia | Tipo | Descripción |
|---|---|---|
| `vInventory` | `cInventory` | Inventario global del jugador. |
| `InventorySorter` | `cIDSorter` | Buscador de items en el inventario. |
| `LObjectSorter` | `cIDSorter` | Buscador de items en la lista global `LObject`. |
| `SLObject` | `list` | `LObject` ordenado por ID. |

---

## Funciones Globales

### `setup_player() -> cPlayer`

Solicita el nombre al usuario por consola y retorna una nueva instancia de `cPlayer` sin arma equipada.

### `Companion_Updater(player)`

Actualiza la lista de compañeros del jugador (actualmente retorna una lista pero no la usa).

### `enemy_loader(data_npc, return_single=False)`

Carga enemigos desde datos JSON y crea instancias de `cEnemy`. Normaliza los arrays de daño y defensa al formato de 5 valores. Retorna una lista o un enemigo aleatorio si `return_single=True`.

---

## Sistema de Daño Multi-tipo

Todos los daños y defensas usan arrays de 5 posiciones:

```
Índice:  0          1         2        3           4
Tipo:    Físico     Térmico   Tierra   Eléctrico   Profundo
```

**Regla especial del tipo Profundo (Deep):** El daño siempre es el valor máximo (sin aleatoriedad).

---

## Ejemplos de Uso

### Crear un jugador

```python
from Modules.setup import setup_player, vInventory

player = setup_player()
# → Solicita nombre por consola
# → Retorna cPlayer(name, weapon=None)
```

### Crear un arma manualmente

```python
from Modules.setup import cWeapon

espada = cWeapon(
    ID=1,
    name="Espada de Hierro",
    DMG=[15, 0, 0, 0, 0],
    DMG_min=[8, 0, 0, 0],
    Gold_Cost=50,
    Description="Una espada básica"
)
```

### Cargar enemigos desde JSON

```python
from Modules.setup import enemy_loader
from Modules.Loader import Read

data = Read("Data/DataStats.json", "Forest", None)
enemies = enemy_loader(data)
enemy = enemies[0]
```

---

## Notas y Referencias

- **`@dataclass`:** [https://docs.python.org/3/library/dataclasses.html](https://docs.python.org/3/library/dataclasses.html)
- **`random.randint(a, b)`:** Genera un entero aleatorio entre a y b (incluidos). [https://docs.python.org/3/library/random.html#random.randint](https://docs.python.org/3/library/random.html#random.randint)
- **`setattr(obj, name, value)`:** Asigna dinámicamente un atributo a un objeto. [https://docs.python.org/3/library/functions.html#setattr](https://docs.python.org/3/library/functions.html#setattr)
- El atributo `MDEF` está marcado como "pendiente de eliminación" en los comentarios del código.
- La clase `cFarm` en `setup.py` es una versión placeholder. Para la lógica de recolección real, referirse a `GatheringMasterySystem.py`.
