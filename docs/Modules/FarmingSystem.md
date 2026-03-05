# `FarmingSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `Skill`](#clase-skill)
3. [Clase `Farm`](#clase-farm)
4. [Estado Actual y Notas de Desarrollo](#estado-actual-y-notas-de-desarrollo)
5. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`FarmingSystem.py` es un módulo en desarrollo que define la estructura base para el sistema de **skills de recolección y habilidades de combate** del jugador. Utiliza `dataclass` para la clase `Skill` y define las listas de skills disponibles.

> **Nota:** Este archivo es una versión simplificada/preliminar. El sistema de recolección completo con lógica funcional se encuentra en `GatheringMasterySystem.py`.

---

## Clase `Skill`

Dataclass que representa una habilidad individual del jugador con su progresión de experiencia.

Decorada con `@dataclass`, por lo que los atributos tienen valores por defecto y el `__init__` se genera automáticamente.

### Atributos

| Atributo | Tipo | Valor por Defecto | Descripción |
|---|---|---|---|
| `exp_total` | `float` | `0.0` | Experiencia total acumulada en esta skill. |
| `exp_to_next_lvl` | `float` | `100.0` | Experiencia necesaria para subir al siguiente nivel. |
| `current_level` | `int` | `1` | Nivel actual de la habilidad. |

```python
from dataclasses import dataclass

@dataclass
class Skill:
    exp_total: float = 0.0
    exp_to_next_lvl: float = 100.0
    current_level: int = 1
```

---

## Clase `Farm`

Gestor de todas las skills del jugador. Inicializa diccionarios de habilidades de recolección y de armas.

### Atributos de clase (listas de skills disponibles)

#### `GATHERING_SKILLS`

```python
GATHERING_SKILLS = ["mining", "lumberjack", "fishing", "herbs", "hunting", "astrology"]
```

Lista de habilidades de recolección disponibles.

| Skill | Descripción |
|---|---|
| `mining` | Minería de recursos. |
| `lumberjack` | Tala de árboles. |
| `fishing` | Pesca. |
| `herbs` | Recolección de plantas/hierbas. |
| `hunting` | Caza de animales. |
| `astrology` | Habilidad especial de astrología. |

#### `WEAPON_SKILLS`

```python
WEAPON_SKILLS = ["swords", "electric_wands", "thermal_wands", "earth_wands",
                 "void_staffs", "shields", "blessed_ornament", "ornament", "cursed_ornament"]
```

Lista de habilidades de armas disponibles.

| Skill | Descripción |
|---|---|
| `swords` | Habilidad con espadas. |
| `electric_wands` | Varas eléctricas. |
| `thermal_wands` | Varas térmicas. |
| `earth_wands` | Varas de tierra. |
| `void_staffs` | Bastones del vacío. |
| `shields` | Escudos. |
| `blessed_ornament` | Ornamento bendecido. |
| `ornament` | Ornamento neutro. |
| `cursed_ornament` | Ornamento maldito. |

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `gathering` | `Dict[str, Skill]` | Diccionario que mapea cada skill de recolección a su objeto `Skill`. |
| `weapons` | `Dict[str, Skill]` | Diccionario que mapea cada skill de arma a su objeto `Skill`. |

### Método `gather(Skill)` *(incompleto)*

```python
def gather(self, Skill):
    # Sin implementación (cuerpo vacío)
```

Este método está declarado pero **no tiene implementación** en este archivo. La versión funcional se encuentra en `GatheringMasterySystem.py` dentro de la clase `cFarm`.

---

## Estado Actual y Notas de Desarrollo

Este archivo parece ser una versión anterior o simplificada del sistema. Las diferencias con `GatheringMasterySystem.py` son:

| Aspecto | `FarmingSystem.py` | `GatheringMasterySystem.py` |
|---|---|---|
| Clase de skill | `Skill` (dataclass) | `cSkill` (clase normal) |
| Clase de farm | `Farm` | `cFarm` |
| `gather()` | Sin implementar | Implementado con barra de progreso |
| Instancia global | No | `Gathering_init = cFarm()` |

> **Recomendación de desarrollo:** Si vas a extender el sistema, usa `GatheringMasterySystem.py` como base. Este archivo puede usarse como referencia de la estructura de datos.

---

## Notas y Referencias

- **`@dataclass`:** Decorador de Python que genera automáticamente `__init__`, `__repr__` y otros métodos especiales. Documentación: [https://docs.python.org/3/library/dataclasses.html](https://docs.python.org/3/library/dataclasses.html)
- **`Dict` de `typing`:** Hint de tipo para diccionarios. En Python 3.9+ se puede usar directamente `dict[str, Skill]`. Documentación: [https://docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html)
- **Sistemas de skills en RPGs:** El diseño es similar al de juegos como *Runescape* o *Hypixel Skyblock*, donde cada acción de recolección contribuye a una skill separada.
