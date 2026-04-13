# `MasterySystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Constantes](#constantes)
3. [Clase `MasterySkill`](#clase-masteryskill)
4. [Clase `MasteryManager`](#clase-masterymanager)
5. [Función Helper `ensure_mastery`](#función-helper-ensure_mastery)
6. [Ejemplos de Uso](#ejemplos-de-uso)
7. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`MasterySystem.py` implementa el sistema de **Mastery para Armas y Armaduras**. Cada tipo de arma y cada tipo de armadura tiene su propio nivel de maestría que sube conforme el jugador combate, otorgando bonificaciones permanentes de daño o reducción de daño.

**Mecánica:**
- **Mastery de Armas:** sube cuando el jugador hace daño. `EXP = daño_hecho / 35`.
- **Mastery de Armaduras:** sube cuando el jugador recibe daño. `EXP = daño_recibido / 35`.

El `MasteryManager` se guarda en `player.mastery_manager` y se serializa/deserializa con el sistema de guardado (`SaveSystem.py`).

---

## Constantes

| Constante | Valor | Descripción |
|---|---|---|
| `MASTERY_MAX_LEVEL` | `20` | Nivel máximo de cualquier mastery. |
| `MASTERY_EXP_BASE` | `100.0` | EXP base para el nivel 1 → 2. |
| `MASTERY_EXP_GROWTH` | `1.5` | Multiplicador de EXP por nivel. |
| `WEAPON_BONUS_PER_LEVEL` | `0.5` | % de daño extra por nivel de arma. |
| `ARMOR_BONUS_PER_LEVEL` | `0.25` | % de reducción de daño por nivel de armadura. |

**Tipos de arma reconocidos (`WEAPON_TYPES`):**

```python
["Swords", "Staffs", "Hammers", "Spears", "Daggers", "Bows", "Generic"]
```

**Tipos de armadura reconocidos (`ARMOR_TYPES`):**

```python
["Helmet", "Armor", "Boots", "Ring"]
```

---

## Clase `MasterySkill`

Representa el nivel de mastery para un tipo de arma o armadura.

### Constructor

```python
MasterySkill(name: str, skill_type: str)
```

| Parámetro | Descripción |
|---|---|
| `name` | Nombre legible (ej: `"Espadas"`, `"Armaduras"`). |
| `skill_type` | `"weapon"` o `"armor"`. |

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `name` | `str` | Nombre legible del mastery. |
| `skill_type` | `str` | Tipo: `"weapon"` o `"armor"`. |
| `current_level` | `int` | Nivel actual (1–20). |
| `exp_current` | `float` | EXP acumulada en el nivel actual. |
| `exp_to_next` | `float` | EXP necesaria para subir al siguiente nivel. |

### Métodos

#### `add_exp(amount: float) -> int`

Agrega EXP al mastery. Maneja múltiples subidas de nivel en una sola llamada. Al llegar al nivel máximo, la EXP se congela en 0. Retorna el número de niveles subidos.

**Fórmula de EXP requerida:**

```python
exp_to_next = MASTERY_EXP_BASE * (MASTERY_EXP_GROWTH ** (nivel_actual - 1))
```

#### `bonus_percent` *(property)*

Retorna el bonus total en porcentaje según nivel:
- Armas: `(nivel - 1) * 0.5%`
- Armaduras: `(nivel - 1) * 0.25%`

#### `bonus_multiplier` *(property)*

Retorna el multiplicador `1.0 + bonus_percent / 100`. Listo para multiplicar directamente contra el daño o la reducción.

#### `to_dict() -> dict`

Serializa el mastery a diccionario para guardado.

#### `from_dict(data: dict) -> MasterySkill` *(classmethod)*

Reconstruye una instancia desde un diccionario guardado.

---

## Clase `MasteryManager`

Gestiona todos los masteries de un jugador. Se instancia una vez y se guarda en `player.mastery_manager`.

### Nombres legibles

```python
WEAPON_NAMES = {
    "Swords": "Espadas", "Staffs": "Bastones", "Hammers": "Martillos",
    "Spears": "Lanzas",  "Daggers": "Dagas",   "Bows": "Arcos",
    "Generic": "Armas Genéricas",
}
ARMOR_NAMES = {
    "Helmet": "Cascos", "Armor": "Armaduras",
    "Boots": "Botas",   "Ring": "Anillos",
}
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `weapon_masteries` | `Dict[str, MasterySkill]` | Un `MasterySkill` por cada tipo de arma. |
| `armor_masteries` | `Dict[str, MasterySkill]` | Un `MasterySkill` por cada tipo de armadura. |

### Métodos

#### `get_weapon_type(weapon) -> str` *(staticmethod)*

Determina el tipo de mastery de un arma. Busca en orden:

1. Atributo `item_category` (ej: `"TYPE:Swords"`) → extrae el tipo quitando el prefijo.
2. Atributo `category`.
3. Infiere del nombre del arma (palabras clave en español e inglés).
4. Retorna `"Generic"` si no puede determinar.

---

#### `register_damage_dealt(player, damage_total: float) -> Optional[str]`

Llamar cuando el jugador hace daño. Agrega `damage_total / 35` EXP al mastery del arma equipada. Retorna un mensaje de level-up si subió de nivel, `None` si no.

```python
msg = mm.register_damage_dealt(player, 120.0)
# → "⚔️  Mastery de Espadas subió a Nv.3! (+1.0% daño)"
```

---

#### `register_damage_received(player, damage_total: float) -> Optional[str]`

Llamar cuando el jugador recibe daño. Agrega EXP a **todos** los masteries de armadura de los slots equipados (sin repetir tipos). Retorna mensajes de level-up concatenados, o `None`.

Los slots mapeados:

| Slot del jugador | Tipo de armadura |
|---|---|
| `helmet` | `Helmet` |
| `armor` | `Armor` |
| `boots` | `Boots` |
| `ring`, `ring2`, `ring3` | `Ring` (cuenta una sola vez) |

---

#### `get_weapon_damage_multiplier(player) -> float`

Retorna el multiplicador de daño basado en el mastery del arma equipada. `1.0` = sin bonus.

---

#### `get_armor_damage_reduction_bonus(player, slot: str) -> float`

Retorna el % extra de reducción para un slot específico, en decimal (0.0–1.0).

---

#### `get_total_armor_reduction_bonus(player) -> float`

Retorna el promedio de bonus de todas las armaduras equipadas (0.0–1.0).

---

#### `to_dict() -> dict`

Serializa todos los masteries a diccionario.

#### `from_dict(data: dict) -> MasteryManager` *(classmethod)*

Reconstruye el manager desde un diccionario guardado. Compatibilidad con saves anteriores: los tipos no encontrados se inicializan en nivel 1.

---

#### `display_masteries(console)`

Muestra todos los masteries en una tabla Rich con barras de progreso, nivel, EXP y bonus.

---

## Función Helper `ensure_mastery`

```python
ensure_mastery(player) -> MasteryManager
```

Garantiza que `player.mastery_manager` exista. Si no existe o es `None`, crea una nueva instancia de `MasteryManager`. Retorna el manager.

Debe llamarse antes de usar el sistema de mastery para garantizar compatibilidad con saves antiguos.

```python
from Modules.MasterySystem import ensure_mastery

mm = ensure_mastery(player)
mm.register_damage_dealt(player, damage)
```

---

## Ejemplos de Uso

### Registrar daño en combate (MainGame.py)

```python
from Modules.MasterySystem import ensure_mastery

# Al atacar
damage_dealt = player.attack(enemy)
mm = ensure_mastery(player)
lvlup_msg = mm.register_damage_dealt(player, float(damage_dealt))
if lvlup_msg:
    console.print(f"[bold green]{lvlup_msg}[/bold green]")

# Al recibir daño
damage_taken = enemy.attack(player)
lvlup_msg = mm.register_damage_received(player, float(damage_taken))
if lvlup_msg:
    console.print(f"[bold cyan]{lvlup_msg}[/bold cyan]")
```

### Consultar bonus de arma equipada

```python
mm = ensure_mastery(player)
multiplier = mm.get_weapon_damage_multiplier(player)
final_damage = base_damage * multiplier
```

### Mostrar tabla de masteries

```python
from rich.console import Console
mm = ensure_mastery(player)
mm.display_masteries(Console())
```

### Guardado y carga (integrado con SaveSystem)

```python
# Serializar (en build_save_data)
mastery_dict = player.mastery_manager.to_dict()

# Deserializar (en _restore_mastery)
player.mastery_manager = MasteryManager.from_dict(mastery_dict)
```

---

## Notas y Referencias

- El mastery se calcula sobre el **daño total** de un golpe, no por tipo de daño individual.
- Los tres slots de anillo (`ring`, `ring2`, `ring3`) contribuyen al mismo tipo `Ring`, pero la EXP solo se otorga una vez por combate para evitar triplicarla.
- **`math`:** Se importa pero no se usa actualmente en la versión actual (puede ser residuo de versiones anteriores).
- La integración con `SaveSystem.py` es completa: `to_dict`/`from_dict` permiten persistir el progreso entre sesiones.
