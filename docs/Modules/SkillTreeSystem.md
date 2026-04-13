# `SkillTreeSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `SkillTreeNode`](#clase-skilltreenode)
3. [Clase `SkillTreeManager`](#clase-skilltreemanager)
4. [Instancia Global](#instancia-global)
5. [Ejemplos de Uso](#ejemplos-de-uso)
6. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`SkillTreeSystem.py` implementa el **árbol de habilidades** del jugador. Los nodos del árbol se cargan desde `Data/DataSkillTree.json` y pueden desbloquearse o mejorarse gastando **puntos de nivel** (`player.lvlPoints`).

El árbol se visualiza en la consola usando `rich.Tree`, mostrando el estado de cada nodo (bloqueado, disponible, parcialmente desbloqueado, máximo). La navegación del menú de desbloqueo usa `interactive_menu_select`.

---

## Clase `SkillTreeNode`

Representa un nodo individual del árbol de habilidades.

### Constructor

```python
SkillTreeNode(node_id: str, data: Dict[str, Any])
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `node_id` | `str` | Identificador único del nodo. |
| `name` | `str` | Nombre visible (ej: `"Golpe Poderoso"`). |
| `desc` | `str` | Descripción del efecto. |
| `base_cost` | `int` | Costo en puntos para el nivel 1. |
| `cost_increment` | `int` | Puntos adicionales por cada nivel subsiguiente. |
| `max_level` | `int` | Nivel máximo del nodo (default `1`). |
| `requires` | `List[str]` | Lista de `node_id`s que deben estar desbloqueados primero. |
| `effects` | `dict` | Efectos aplicados al desbloquear/mejorar. |

### Métodos

#### `get_current_cost(current_level: int) -> int`

Retorna el costo para subir desde `current_level` al siguiente nivel.

```python
costo = node.get_current_cost(2)
# base_cost + (2 * cost_increment)
```

---

## Clase `SkillTreeManager`

Gestor principal del árbol de habilidades.

### Constructor

```python
SkillTreeManager()
```

Llama automáticamente a `load_tree()`.

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `nodes` | `Dict[str, SkillTreeNode]` | Diccionario de todos los nodos cargados. |

### Métodos

#### `load_tree()`

Carga los nodos desde `Data/DataSkillTree.json`. Cada clave del JSON bajo `"nodes"` se convierte en un `SkillTreeNode`.

**Estructura esperada en `DataSkillTree.json`:**

```json
{
  "nodes": {
    "root_node": {
      "name": "Origen",
      "desc": "El punto de partida.",
      "cost": 0,
      "cost_increment": 0,
      "max_level": 1,
      "requires": [],
      "effects": {}
    },
    "power_strike": {
      "name": "Golpe Poderoso",
      "desc": "+5% de daño físico.",
      "cost": 1,
      "cost_increment": 1,
      "max_level": 3,
      "requires": ["root_node"],
      "effects": { "AGI": 5 }
    }
  }
}
```

---

#### `build_rich_tree(player) -> Tree`

Construye y retorna el árbol visual de `rich.Tree`. Muestra el estado de cada nodo con iconos:

| Ícono | Significado |
|---|---|
| `❇️` | Nodo en nivel máximo. |
| `🔼` (verde) | Puede mejorarse (puntos suficientes). |
| `🔴` | Puede mejorarse pero sin puntos suficientes. |
| `🟢` | Disponible para desbloquear (puntos suficientes). |
| `🔴` | Disponible para desbloquear pero sin puntos. |
| `🔒` | Bloqueado (requiere nodos previos). |

Los nodos raíz (sin `requires`) siempre se garantizan como poseídos al nivel 1 al construir el árbol.

---

#### `apply_node_effects(player, node: SkillTreeNode)`

Aplica los efectos de un nodo al jugador de forma **permanente** al desbloquearlo/mejorarlo.

**Stats soportados:**

| Key en `effects` | Tipo del valor | Comportamiento |
|---|---|---|
| `Health_max` | `int` | Aumenta HP máximo; si positivo, también HP actual. |
| `Mana_max` | `int` | Aumenta MP máximo; si positivo, también MP actual. |
| `DEF` | `list[5]` | Suma cada elemento al array `player.DEF`. |
| `DefensePercentage` | `list[5]` | Suma cada elemento al array `player.DefensePercentage`. |
| Cualquier otro atributo | `numeric` | `setattr(player, stat, getattr(player, stat) + value)`. |

---

#### `open_menu(player)`

Abre el menú interactivo del árbol de habilidades. Muestra el árbol visual y la lista de nodos disponibles para desbloquear o mejorar.

**Flujo:**

1. Garantiza que `player.skills_purchased` sea un `dict` (migración de formato lista si es necesario).
2. Limpia pantalla y muestra el árbol visual con los puntos disponibles en el panel.
3. Lista opciones desbloqueables (nodos cuyos requisitos están cumplidos y no están al máximo).
4. El jugador selecciona con flechas + Enter.
5. Si tiene puntos suficientes: descuenta puntos, actualiza `player.skills_purchased[node_id]` y aplica efectos.

**Validación de retrocompatibilidad:** Si `player.skills_purchased` es una lista (formato antiguo), se convierte a diccionario automáticamente.

---

## Instancia Global

```python
skill_tree_manager = SkillTreeManager()
```

Se crea una instancia global pre-cargada al importar el módulo. Todos los módulos que necesiten el árbol deben usar esta instancia o acceder a ella mediante `from Modules.SkillTreeSystem import skill_tree_manager`.

---

## Ejemplos de Uso

### Abrir el menú desde el juego

```python
from Modules.SkillTreeSystem import skill_tree_manager

# Llamado desde MainGame cuando el jugador selecciona opción 14
skill_tree_manager.open_menu(player)
```

> En `MainGame.py` esto se hace a través de `player.Points()`, que delega en `skill_tree_manager.open_menu(player)`.

### Verificar si un nodo está desbloqueado

```python
nivel = player.skills_purchased.get("power_strike", 0)
if nivel >= 1:
    print("Golpe Poderoso desbloqueado")
```

### Aplicar efectos manualmente

```python
from Modules.SkillTreeSystem import skill_tree_manager

node = skill_tree_manager.nodes.get("power_strike")
if node:
    skill_tree_manager.apply_node_effects(player, node)
```

### Estructura de `player.skills_purchased`

```python
# Dict de {node_id: nivel_actual}
player.skills_purchased = {
    "root_node":     1,
    "power_strike":  2,
    "mana_boost":    1,
}
```

---

## Notas y Referencias

- Los nodos raíz (sin `requires`) se auto-poseen al nivel 1. No es necesario desbloquearlos manualmente.
- El costo total acumulado para un nodo de `max_level=3` con `base_cost=1` y `cost_increment=1` sería: `1 + 2 + 3 = 6` puntos.
- Los efectos se aplican **cada vez que se sube de nivel**, por lo que un nodo de nivel 3 aplica sus efectos 3 veces en total.
- **`rich.Tree`:** Componente para mostrar árboles jerárquicos. [https://rich.readthedocs.io/en/stable/tree.html](https://rich.readthedocs.io/en/stable/tree.html)
- **`rich.Panel`:** [https://rich.readthedocs.io/en/stable/panel.html](https://rich.readthedocs.io/en/stable/panel.html)
