# `EnchantmentSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `cEnchantment`](#clase-cenchantment)
3. [Funciones Helper](#funciones-helper)
4. [Ejemplos de Uso](#ejemplos-de-uso)
5. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`EnchantmentSystem.py` define el sistema de **encantamientos** del juego. Los encantamientos son mejoras que se aplican a un item específico (arma, equipamiento o herramienta) para modificar sus estadísticas o añadir efectos especiales.

Cada encantamiento es compatible solo con un tipo de item, definido por su atributo `TargetClass`.

---

## Clase `cEnchantment`

Clase universal que representa cualquier encantamiento del juego, independientemente del tipo de item al que se aplica.

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `ENCHANT_ID` | `int` | ID único del encantamiento. |
| `Name` | `str` | Nombre del encantamiento (ej: `"Sharpness"`). |
| `TargetClass` | `str` | Clase de item compatible: `"cWeapon"`, `"cEquippableItems"` o `"cTool"`. |
| `Description` | `str` | Texto descriptivo del efecto. |
| `Rarity` | `str` | Rareza: `"Common"`, `"Uncommon"`, `"Rare"`, `"Epic"`, `"Legendary"`. |
| `Effects` | `dict` | Diccionario con los efectos del encantamiento y sus valores. |
| `MaxLevel` | `int` | Nivel máximo del encantamiento (para los que se pueden mejorar). Default: `1`. |
| `GoldCost` | `int` | Costo en oro para aplicar el encantamiento. |
| `current_level` | `int` | Nivel actual cuando está aplicado (empieza en `1`). |

### Métodos

#### `can_apply_to(item) -> bool`

Verifica si el encantamiento es compatible con el item recibido, comparando `type(item).__name__` con `self.TargetClass`.

```python
enchant.can_apply_to(my_sword)  # True si my_sword es instancia de cWeapon
```

---

#### `apply_to_weapon(weapon) -> Dict[str, Any]`

Aplica los efectos del encantamiento a un arma. Solo funciona si `TargetClass == "cWeapon"`.

**Effects disponibles para armas:**

| Clave en `Effects` | Tipo | Descripción |
|---|---|---|
| `dmg_bonus` | `float` o `list[5]` | Bonus porcentual (float) o plano por tipo (lista). |
| `dmg_min_bonus` | `float` o `list[4]` | Bonus al daño mínimo. |
| `crit_chance` | `float` | Probabilidad de golpe crítico (0.0–1.0). |
| `crit_damage` | `float` | Multiplicador de daño crítico (ej: `2.0` = 200%). |
| `lifesteal` | `float` | % de daño robado como vida. |
| `elemental_boost` | `dict` | Boost por tipo elemental, ej: `{"thermal": 1.2}`. |
| `special_effect` | `str` | Nombre de un efecto especial. |

---

#### `apply_to_equipment(equipment) -> Dict[str, Any]`

Aplica efectos al equipamiento. Solo funciona si `TargetClass == "cEquippableItems"`.

**Effects disponibles para equipamiento:**

| Clave en `Effects` | Tipo | Descripción |
|---|---|---|
| `defense_bonus` | `float` o `list[5]` | Bonus de defensa porcentual o plano. |
| `health_bonus` | `int` | HP extra. |
| `health_regen` | `float` | HP regenerado por segundo/turno. |
| `damage_reduction` | `float` | % de reducción de daño recibido (0.0–1.0). |
| `elemental_resistance` | `dict` | Resistencias por tipo elemental. |
| `reflect_damage` | `float` | % de daño reflejado al atacante. |
| `thorns` | `int` | Daño fijo devuelto al atacante. |
| `special_effect` | `str` | Nombre de un efecto especial. |

---

#### `apply_to_tool(tool) -> Dict[str, Any]`

Aplica efectos a herramientas. Solo funciona si `TargetClass == "cTool"`.

**Effects disponibles para herramientas:**

| Clave en `Effects` | Tipo | Descripción |
|---|---|---|
| `efficiency_bonus` | `float` | Multiplicador de eficiencia. |
| `durability_bonus` | `float` | Multiplicador de durabilidad. |
| `fortune` | `int` | Nivel de Fortune (drops extra). |
| `auto_smelt` | `bool` | Fundir automáticamente al recolectar. |
| `silk_touch` | `bool` | Obtener bloque en estado natural. |
| `speed_bonus` | `float` | Velocidad de recolección extra. |
| `exp_bonus` | `float` | Multiplicador de EXP de recolección. |
| `special_effect` | `str` | Nombre de un efecto especial. |

---

#### `get_display_name() -> str`

Retorna el nombre del encantamiento. Si `MaxLevel > 1`, añade el numeral romano del nivel actual.

```
"Sharpness III"  # Si current_level == 3 y MaxLevel > 1
"Fire Aspect"    # Si MaxLevel == 1
```

#### `get_roman_numeral(num) -> str` *(static)*

Convierte un número del 1 al 5 a su numeral romano. Usado internamente por `get_display_name()`.

#### `get_rarity_color() -> str`

Retorna el color asociado a la rareza del encantamiento:

| Rareza | Color |
|---|---|
| Common | white |
| Uncommon | green |
| Rare | blue |
| Epic | purple |
| Legendary | gold |

---

## Funciones Helper

### `load_enchantment_from_json(enchant_data: Dict) -> cEnchantment`

Crea un objeto `cEnchantment` a partir de un diccionario (normalmente cargado desde JSON). Espera las claves: `ENCHANT_ID`, `Name`, `TargetClass`, `Description`, `Rarity`, `Effects`, y opcionalmente `MaxLevel` y `GoldCost`.

```python
data = {
    "ENCHANT_ID": 1,
    "Name": "Sharpness",
    "TargetClass": "cWeapon",
    "Description": "Aumenta el daño físico",
    "Rarity": "Common",
    "Effects": {"dmg_bonus": [5, 0, 0, 0, 0]},
    "MaxLevel": 5,
    "GoldCost": 100
}
enchant = load_enchantment_from_json(data)
```

---

### `get_enchantments_for_item(item, enchantments_list) -> List[cEnchantment]`

Filtra la lista de encantamientos y retorna solo los que son compatibles con el item dado, usando `enchant.can_apply_to(item)`.

```python
# Obtener solo los encantamientos aplicables a esta espada
compatible = get_enchantments_for_item(my_sword, all_enchantments)
```

---

## Ejemplos de Uso

### Crear un encantamiento para arma

```python
from Modules.EnchantmentSystem import cEnchantment

sharpness = cEnchantment(
    ENCHANT_ID=1,
    Name="Sharpness",
    TargetClass="cWeapon",
    Description="Aumenta el daño físico del arma",
    Rarity="Common",
    Effects={
        "dmg_bonus": [5, 0, 0, 0, 0],  # +5 daño físico
        "dmg_min_bonus": [3, 0, 0, 0]
    },
    MaxLevel=5,
    GoldCost=100
)

print(sharpness.get_display_name())   # "Sharpness I"
print(sharpness.get_rarity_color())   # "white"

# Subir nivel
sharpness.current_level = 3
print(sharpness.get_display_name())   # "Sharpness III"
```

### Filtrar encantamientos compatibles

```python
from Modules.EnchantmentSystem import get_enchantments_for_item

all_enchants = [sharpness, fire_aspect, protection]  # Lista de cEnchantment
sword_enchants = get_enchantments_for_item(my_sword, all_enchants)
# Retorna solo los que tienen TargetClass == "cWeapon"
```

### Aplicar encantamiento a equipo y ver modificadores

```python
protection = cEnchantment(
    ENCHANT_ID=10,
    Name="Protection",
    TargetClass="cEquippableItems",
    Description="Reduce el daño recibido",
    Rarity="Uncommon",
    Effects={"damage_reduction": 0.10},  # 10% de reducción
    GoldCost=200
)

mods = protection.apply_to_equipment(my_armor)
print(mods)
# {"damage_reduction": 0.1}
```

---

## Notas y Referencias

- El sistema actualmente no persiste los encantamientos al JSON; se construye en memoria. Una implementación futura podría integrar el `ItemManager` para guardar encantamientos aplicados.
- La numeración romana está limitada del I al V. Si `MaxLevel > 5`, se mostrará el número directamente.
- **Referencia sobre sistemas de encantamientos en juegos:** El diseño está inspirado en el sistema de Minecraft, donde los encantamientos tienen niveles y compatibilidades por tipo de item.
- **`type(item).__name__`:** Forma de obtener el nombre de la clase de un objeto en Python. Documentación: [https://docs.python.org/3/library/functions.html#type](https://docs.python.org/3/library/functions.html#type)
