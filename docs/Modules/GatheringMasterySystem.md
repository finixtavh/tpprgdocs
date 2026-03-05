# `GatheringMasterySystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Variables Globales](#variables-globales)
3. [Clase `cSkill`](#clase-cskill)
4. [Clase `cFarm`](#clase-cfarm)
5. [Clase `cMaterial`](#clase-cmaterial)
6. [Clase `cTool`](#clase-ctool)
7. [Ejemplos de Uso](#ejemplos-de-uso)
8. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`GatheringMasterySystem.py` implementa el sistema completo de **recolección de recursos** del juego. Incluye la lógica de recolección con barra de progreso visual (usando la librería `rich`), sistema de skills con niveles, materiales y herramientas.

Cuando el jugador recolecta un material, el tiempo necesario depende de:

- La dureza del material (`Hardness`).
- La eficiencia de la herramienta (`Efficiency`).
- El nivel del jugador en la skill correspondiente.
- Buffs activos del jugador.

---

## Variables Globales

| Variable | Tipo | Descripción |
|---|---|---|
| `Gathering_init` | `cFarm` | Instancia global del sistema de recolección. Se crea al importar el módulo. |

```python
Gathering_init = cFarm()
```

---

## Clase `cSkill`

Representa una habilidad individual de recolección o arma con su sistema de progresión.

> **Nota:** A diferencia de `FarmingSystem.py`, esta clase **no usa `@dataclass`**; los atributos de clase actúan como valores por defecto compartidos entre instancias. Esto es una particularidad del código que puede causar comportamientos inesperados si se modifican directamente los atributos de clase.

### Atributos

| Atributo | Tipo | Valor por Defecto | Descripción |
|---|---|---|---|
| `exp_total` | `float` | `0.0` | Experiencia total acumulada. |
| `exp_to_next_lvl` | `float` | `100.0` | Experiencia necesaria para el próximo nivel. |
| `current_level` | `int` | `1` | Nivel actual de la habilidad. |

---

## Clase `cFarm`

Gestor de todas las skills del jugador y punto de entrada para iniciar recolecciones.

### Atributos de clase

```python
GATHERING_SKILLS = ["mining", "lumberjack", "fishing", "herbs", "hunting", "astrology"]

WEAPON_SKILLS = ["swords", "electric_wands", "thermal_wands", "earth_wands",
                 "void_staffs", "shields", "blessed_ornament", "ornament", "cursed_ornament"]
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `gathering` | `Dict[str, cSkill]` | Skills de recolección, inicializadas a nivel 1. |
| `weapons` | `Dict[str, cSkill]` | Skills de armas, inicializadas a nivel 1. |

### Métodos

#### `gather(Material, Tool, vInventory, buffs=None)`

Ejecuta la acción de recolección. Muestra una barra de progreso animada y al finalizar otorga EXP a la skill correspondiente.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `Material` | `cMaterial` | El material a recolectar. |
| `Tool` | `cTool` o `None` | La herramienta usada. Si es `None`, eficiencia = 1.0. |
| `vInventory` | `cInventory` | El inventario del jugador (para añadir el material). |
| `buffs` | `list[float]` | Lista de multiplicadores de buff (ej: `[1.1, 1.2]`). |

**Cálculo de poder de recolección:**

```python
Gathering_Skill_Bonus = 1 + (nivel_actual * 0.05)
total_modifier = producto_de_todos_los_buffs
tool_eff = Tool.Efficiency  # 1.0 si no hay herramienta

power = tool_eff * Gathering_Skill_Bonus * total_modifier
```

**Cálculo de EXP ganada:**

```python
exp_ganada = Material.Hardness / 13
```

**Barra de progreso:**

Usa `rich.progress.Progress` con `BarColumn` y `TimeRemainingColumn`. El progreso avanza en `power` unidades cada 0.1 segundos hasta alcanzar `Material.Hardness`.

---

## Clase `cMaterial`

Representa un material recolectable del mundo del juego.

### Constructor

```python
cMaterial(MATERIAL_ID, TypeStr, Name, Hardness, SkillName)
```

### Atributos

| Atributo | Tipo | Descripción |
|---|---|---|
| `MATERIAL_ID` | `int` | ID único del material. |
| `Material_Type` | `str` | Tipo del material con prefijo: `"TYPE_M:" + TypeStr`. |
| `Name` | `str` | Nombre del material (ej: `"Hierro"`, `"Roble"`). |
| `Hardness` | `float` | Dureza: determina cuánto tarda en recolectarse. |
| `SkillName` | `str` | Nombre de la skill que se usa para recolectarlo (debe coincidir con una key en `cFarm.gathering`). |

---

## Clase `cTool`

Representa una herramienta de recolección.

### Constructor

```python
cTool(STATIC_ID, TypeStr, Name, Desc, Efficiency, Enchantments)
```

### Atributos

| Atributo | Tipo | Descripción |
|---|---|---|
| `STATIC_ID` | `int` | ID único de la herramienta. |
| `ToolTypeStr` | `str` | Tipo de herramienta con prefijo: `"TYPE_M:" + TypeStr`. |
| `Name` | `str` | Nombre de la herramienta. |
| `Desc` | `str` | Descripción. |
| `Efficiency` | `float` | Multiplicador de eficiencia (ej: `2.0` = doble de rápido). |
| `Enchantments` | `List` | Lista de encantamientos aplicados (ver `EnchantmentSystem.py`). |

---

## Ejemplos de Uso

### Recolectar un material básico

```python
from Modules.GatheringMasterySystem import cFarm, cMaterial, cTool, Gathering_init

# Crear material
hierro = cMaterial(
    MATERIAL_ID=1,
    TypeStr="OreBlock",
    Name="Mineral de Hierro",
    Hardness=50.0,
    SkillName="mining"
)

# Crear herramienta
pico = cTool(
    STATIC_ID=101,
    TypeStr="Pickaxe",
    Name="Pico de Hierro",
    Desc="Un pico resistente",
    Efficiency=2.5,
    Enchantments=[]
)

# Recolectar (bloquea hasta terminar, muestra barra)
Gathering_init.gather(hierro, pico, vInventory)
```

### Recolectar con buffs activos

```python
# Los buffs son multiplicadores de eficiencia
buffs = [1.2, 1.1]  # +20% y +10% extra
Gathering_init.gather(hierro, pico, vInventory, buffs=buffs)
```

### Verificar nivel de una skill

```python
nivel_mineria = Gathering_init.gathering["mining"].current_level
exp_actual = Gathering_init.gathering["mining"].exp_total
print(f"Minería: Nivel {nivel_mineria}, EXP: {exp_actual}")
```

---

## Notas y Referencias

- **Barra de progreso con `rich`:** La librería `rich` provee componentes visuales avanzados para la terminal. Documentación: [https://rich.readthedocs.io/en/stable/progress.html](https://rich.readthedocs.io/en/stable/progress.html)
- **Atributos de clase vs. atributos de instancia en Python:** Los atributos definidos en el cuerpo de la clase (sin `self`) son compartidos entre instancias. Esto puede causar bugs si se modifican. Para evitarlo, deben definirse en `__init__`. Referencia: [https://docs.python.org/3/tutorial/classes.html#class-and-instance-variables](https://docs.python.org/3/tutorial/classes.html#class-and-instance-variables)
- **`time.sleep(0.1)`:** Cada ciclo de la barra espera 0.1 segundos. Materiales muy duros pueden tardar varios segundos en recolectarse.
- El nombre de la skill en `cMaterial.SkillName` **debe coincidir exactamente** (en minúsculas) con una clave del diccionario `gathering` de `cFarm`. Si no coincide, se producirá un `KeyError`.
