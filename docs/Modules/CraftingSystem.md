# `CraftingSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `cRecipe`](#clase-crecipe)
3. [Clase `CraftingManager`](#clase-craftingmanager)
4. [Función `open_crafting_menu`](#función-open_crafting_menu)
5. [Ejemplos de Uso](#ejemplos-de-uso)
6. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`CraftingSystem.py` implementa el sistema de **crafteo (forja)** del juego. Permite al jugador combinar materiales del inventario para crear nuevos items, consumiendo los materiales y un costo en oro definidos en cada receta.

Las recetas se cargan desde `Data/DataRecipes.json`. La interfaz visual utiliza `rich` con el mismo estilo de menú interactivo del resto del juego.

---

## Clase `cRecipe`

Representa una receta de crafteo individual.

### Constructor

```python
cRecipe(data: dict)
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `recipe_id` | `any` | ID único de la receta. |
| `name` | `str` | Nombre del item resultante. |
| `result_item_id` | `any` | ID del item producido (debe existir en `DataItems.json`). |
| `result_item_type` | `str` | Tipo del item resultante (ej: `"Weapon"`, `"Tool"`, `"Object"`). |
| `materials_required` | `dict` | Diccionario `{nombre_material: cantidad}`. |
| `description` | `str` | Descripción de la receta. |
| `gold_cost` | `int` | Costo en oro para fabricar. |

### Estructura esperada en `DataRecipes.json`

```json
{
  "recipes": [
    {
      "recipe_id": 1,
      "name": "Espada de Hierro",
      "result_item_id": 2,
      "result_item_type": "Weapon",
      "materials_required": {
        "Mineral de Hierro": 3,
        "Madera": 1
      },
      "description": "Una espada básica forjada en hierro.",
      "gold_cost": 50
    }
  ]
}
```

---

## Clase `CraftingManager`

Gestor de recetas y lógica de crafteo. Se instancia de forma local dentro de `open_crafting_menu` en cada llamada.

### Constructor

```python
CraftingManager()
```

Llama automáticamente a `load_recipes()` al inicializarse.

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `recipes` | `List[cRecipe]` | Lista de todas las recetas cargadas. |

### Métodos

#### `load_recipes()`

Carga las recetas desde `Data/DataRecipes.json`. Si hay error, lo muestra en consola con `rich` en rojo y deja `self.recipes` vacío.

---

#### `count_material(vInventory, material_name: str) -> int`

Cuenta cuántas unidades de un material por nombre existen en el inventario. La comparación es **case-insensitive**.

Busca en `vInventory.Objects` el atributo `Name` o `name` del item.

```python
cantidad = cm.count_material(vInventory, "Mineral de Hierro")
```

---

#### `consume_materials(vInventory, material_name: str, amount: int)`

Elimina `amount` unidades de un material del inventario. Busca items por nombre (case-insensitive) y llama a `vInventory.remove_item(item, 1)` por cada unidad consumida hasta alcanzar la cantidad requerida.

---

## Función `open_crafting_menu`

```python
open_crafting_menu(player, vInventory)
```

Abre el menú interactivo del **Yunque de Crafteo**. Usa `interactive_menu_select` de `input_utils.py` para la navegación.

**Flujo:**

1. Limpia la pantalla y muestra el panel del yunque con el oro actual del jugador.
2. Construye la lista de opciones con todas las recetas disponibles.
3. Para cada receta, muestra los materiales requeridos en color verde (✅ disponible) o rojo (❌ faltante).
4. El jugador selecciona una receta con las flechas + Enter.
5. Se validan el oro y los materiales.
6. Si todo está disponible: descuenta el oro, consume los materiales, crea el item con `item_manager.create_item_object()` y lo añade al inventario.

**Indicadores visuales por receta:**

```
[bold cyan]Espada de Hierro[/]
  └ Costo: [green]50g[/] | [green]Mineral de Hierro (3/3)[/], [red]Madera (0/1)[/]
```

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `player` | `cPlayer` | El jugador (para verificar/descontar oro). |
| `vInventory` | `cInventory` | El inventario del jugador. |

**Casos de error manejados:**

| Situación | Comportamiento |
|---|---|
| Oro insuficiente | Mensaje de error, espera 1.5s, vuelve al menú. |
| Materiales faltantes | Mensaje de error, espera 1.5s, vuelve al menú. |
| Item no encontrado en `DataItems.json` | Mensaje de error en rojo, espera 2.5s. |
| Error al instanciar el item | Mensaje de error, espera 1.5s. |

---

## Ejemplos de Uso

### Abrir el menú de crafteo desde el juego

```python
from Modules.CraftingSystem import open_crafting_menu

open_crafting_menu(player, vInventory)
```

### Verificar materiales manualmente

```python
from Modules.CraftingSystem import CraftingManager

cm = CraftingManager()
cantidad_hierro = cm.count_material(vInventory, "Mineral de Hierro")
print(f"Hierro disponible: {cantidad_hierro}")
```

### Consumir materiales directamente

```python
cm = CraftingManager()
cm.consume_materials(vInventory, "Madera", 2)
# Elimina 2 unidades de "Madera" del inventario
```

---

## Notas y Referencias

- El `CraftingManager` se instancia localmente en cada llamada a `open_crafting_menu`, por lo que las recetas se recargan del JSON en cada apertura del menú. Esto permite modificar `DataRecipes.json` en caliente sin reiniciar el juego.
- Los nombres de materiales en `materials_required` deben coincidir exactamente (ignorando mayúsculas/minúsculas) con el atributo `Name` o `name` de los items en el inventario.
- El `result_item_id` de cada receta debe existir en `DataItems.json` y ser compatible con `item_manager.create_item_object()`.
- **`rich.Panel`:** [https://rich.readthedocs.io/en/stable/panel.html](https://rich.readthedocs.io/en/stable/panel.html)
- **`input_utils.interactive_menu_select`:** Ver `input_utils.md` para documentación completa del sistema de menús interactivos.
