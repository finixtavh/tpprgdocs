# `SkillTreeSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `SkillTreeNode`](#clase-skilltreenode)
3. [Clase `SkillTreeManager`](#clase-skilltreemanager)
4. [Lógica de Costes y Niveles](#lógica-de-costes-y-niveles)
5. [Aplicación de Efectos](#aplicación-de-efectos)
6. [Interfaz Visual](#interfaz-visual)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`SkillTreeSystem.py` gestiona la progresión de habilidades y estadísticas permanentes del **jugador**. Utiliza un sistema de nodos ramificados cargados desde `DataSkillTree.json`. Los puntos de habilidad (`lvlPoints`) se obtienen al subir de nivel en el juego principal.

---

## Clase `SkillTreeNode`

Representa un nodo (o "talento") individual en el árbol.

- `node_id`: ID técnico del nodo.
- `requires`: Lista de IDs de nodos previos necesarios para desbloquear este.
- `max_level`: Número máximo de veces que se puede mejorar este nodo.
- `effects`: Diccionario de cambios permanentes en los atributos del jugador (ej. `{"Health_max": 20}`).

---

## Clase `SkillTreeManager`

Coordina la carga, visualización y compra de habilidades.

### Métodos Principales
- `load_tree()`: Carga la estructura completa desde el JSON.
- `build_rich_tree(player)`: Genera un objeto `Tree` de la librería `Rich` para mostrar visualmente las ramificaciones, estados (bloqueado/desbloqueado) y costes.
- `apply_node_effects(player, node)`: Modifica directamente el objeto `player` aplicando los bonus del nodo.
- `open_menu(player)`: Menú interactivo de selección y compra.

---

## Lógica de Costes y Niveles

El coste de una habilidad no es necesariamente estático. Se calcula mediante la fórmula:
`Costo Actual = base_cost + (current_level * cost_increment)`

Esto permite que habilidades de múltiples niveles (como "Vitalidad") se vuelvan más caras con cada mejora.

---

## Aplicación de Efectos

Cuando se compra una habilidad, el `SkillTreeManager` aplica los cambios al objeto `player`. Soporta:
- **Salud y Maná**: Aumenta el máximo y cura la misma cantidad actual.
- **Defensas**: Incrementa los valores fijos del array `DEF`.
- **Porcentajes**: Incrementa los valores del array `DefensePercentage`.
- **Atributos Dinámicos**: Cualquier otro atributo presente en la clase `cPlayer`.

---

## Interfaz Visual

El sistema utiliza `rich.tree` para renderizar un diagrama jerárquico en la consola:
- 🌟 **Origen**: Nodo raíz inicial.
- 🟢 **Desbloqueable**: Nodo con requisitos cumplidos y puntos suficientes.
- 🔒 **Bloqueado**: Nodo sin los requisitos previos.
- ❇️ **Máximo**: Nodo que ya alcanzó su `max_level`.

---

## Ejemplos de Uso

### Abrir el menú desde el bucle principal

```python
from Modules.ModulesFunctions.SkillTreeSystem import skill_tree_manager

if key == "k":
    skill_tree_manager.open_menu(player)
```

### Consultar si una habilidad fue comprada

```python
# Útil para diálogos o eventos especiales
if "double_jump" in player.skills_purchased:
    print("¡Puedes saltar dos veces!")
```
