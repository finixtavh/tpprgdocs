# `SkillTreeSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Desbloqueo y Construcción Visual](#flujo-de-desbloqueo)

---

## 1. Descripción General
El `SkillTreeSystem.py` gestiona el árbol de habilidades (o talentos) del jugador. Permite gastar los `lvlPoints` (ganados al subir de nivel) en nodos que otorgan bonificaciones permanentes (salud, maná, atributos, etc.). Soporta ramificaciones, donde algunos nodos están bloqueados hasta que se completen los nodos previos ("raíces"). Presenta un menú interactivo en consola construido de forma visualmente jerárquica utilizando `rich.tree`.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere `./Data/DataSkillTree.json` que define el grafo de habilidades, los costos y los requerimientos de ramificación.
- **Librerías TUI**: `rich.tree` para el renderizado jerárquico, `rich.panel` y `rich.console`.
- **Inyecciones Condicionales**: `ModulesUtils.input_utils` (para el menú interactivo seguro `interactive_menu_select`).

---

## 3. Constantes y Variables Globales
- `skill_tree_manager` (`SkillTreeManager`): Instancia global pre-cargada. Expone el gestor como un Singleton de facto importable por cualquier otro script sin necesidad de volver a leer el archivo JSON desde cero.

---

## 4. Clases y Estructuras de Datos

### `SkillTreeNode`
Representa un talento individual (un punto del grafo).

#### Atributos de Instancia
- `self.node_id` (`str`): ID interno.
- `self.name` (`str`), `self.desc` (`str`): UI legible.
- `self.base_cost` (`int`): Costo en `lvlPoints` del primer nivel.
- `self.cost_increment` (`int`): Cuánto encarece la habilidad por cada nivel extra comprado.
- `self.max_level` (`int`): Límite de veces que se puede comprar.
- `self.requires` (`List[str]`): IDs de nodos "padre" que deben poseerse para poder desbloquear este.
- `self.effects` (`Dict`): Diccionario de bonificaciones, ej: `{"Health_max": 50, "AGI": 2}`.

#### Métodos Principales
- `get_current_cost(self, current_level: int) -> int`: Retorna la matemática `base_cost + (current_level * cost_increment)`.

### `SkillTreeManager`
El motor que maneja el árbol global.

#### Atributos de Instancia
- `self.nodes` (`Dict[str, SkillTreeNode]`): Caché de todos los nodos parseados de JSON.

#### Métodos Principales
- `load_tree(self)`: Parsea el JSON y llena el diccionario de `nodes`.
- `build_rich_tree(self, player) -> Tree`: Recrea visualmente el árbol. Identifica los nodos que no tienen `requires` (Raíces). Luego itera recursivamente descubriendo a los "Hijos" que requieran al padre, decorándolos con colores semánticos (Verde si lo tienes, Amarillo si lo maxeaste, Rojo si no tienes puntos, Gris si está bloqueado).
- `apply_node_effects(self, player, node: SkillTreeNode)`: Intérprete de mutación. Lee el diccionario de `effects` y los suma directamente a las estadísticas base del objeto `player`.
- `open_menu(self, player)`: Bucle asíncrono. Muestra el árbol renderizado, filtra solo las opciones viables y delega la compra descontando los `lvlPoints`.

---

## 5. Funciones del Módulo (API)

El módulo exporta directamente su clase de forma global, por lo que las funciones a llamar siempre son:
- `skill_tree_manager.open_menu(player)`

---

## 6. Flujo de Desbloqueo y Construcción Visual

El sistema fue rediseñado de listas a diccionarios en los guardados (`skills_purchased`).
1. Al abrir el menú, si el jugador viene de una partida muy antigua donde `skills_purchased` era una lista, hace una migración automática (`retrocompatibilidad a dict`).
2. Se renderiza un árbol usando la librería `rich`. El árbol detecta dinámicamente las relaciones Padre-Hijo gracias a la propiedad `requires`.
3. El árbol comprueba silenciosamente y forza el desbloqueo a nivel 1 de los nodos raíz (para prevenir que los jugadores nuevos se bloqueen fuera del árbol).
4. El jugador selecciona un nodo de la lista, el sistema cobra los Puntos de Nivel y aplica los modificadores matemáticos (como "Salud +50") **directamente y sin intermediarios** al objeto `Player`, mutando sus variables pasivamente.
