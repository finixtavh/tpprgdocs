# `CraftingSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `CraftingSystem`](#clase-craftingsystem)
3. [Lógica de Recetas](#lógica-de-recetas)
4. [Interfaz: Mesa de Crafteo](#interfaz-mesa-de-crafteo)
5. [Controles](#controles)
6. [ModLoader e Inyecciones](#modloader-e-inyecciones)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`CraftingSystem.py` permite al jugador crear nuevos objetos combinando materiales de su inventario. El sistema es totalmente dinámico, cargando recetas desde `DataRecipes.json` y permitiendo a los mods inyectar sus propias fórmulas.

---

## Clase `CraftingSystem`

Coordina la carga de recetas y el acceso al inventario del jugador.

- `_load_recipes()`: Lee las recetas base y consulta al `ModLoader` por recetas adicionales.
- `recipes`: Almacena la lista de recetas normalizadas (ID, resultado, materiales, coste en oro).

---

## Lógica de Recetas

Cada receta define:
- **Resultado**: ID del ítem generado y cantidad.
- **Ingredientes**: Diccionario de `{item_id: cantidad_requerida}`.
- **Coste de Oro**: (Opcional) Precio adicional por el trabajo de forja.

Cuando el jugador craftea un objeto, el sistema:
1. Verifica si posee todos los materiales (y oro).
2. Los consume del inventario.
3. Utiliza el `ItemManager` para crear el nuevo objeto.
4. Registra el objeto en `player.discovered_items`.

---

## Interfaz: Mesa de Crafteo

### `open_crafting_menu(player, inventory)`
Interfaz dividida en dos paneles:
1. **Panel Izquierdo (Recetas)**: Muestra la lista de fórmulas conocidas. Indica visualmente si están "LISTO" (tienes materiales) o "FALTAN".
2. **Panel Derecho (Detalle)**: Desglose exacto de materiales (ej. `Hierro: 5/10`).

---

## Controles

| Tecla | Acción |
|---|---|
| **Enter** | Craftear una unidad del ítem seleccionado. |
| **Q** | Crafteo múltiple (abre un buffer para introducir cantidad numérica). |
| **S** | Buscar recetas por nombre. |
| **F** | Marcar receta como favorita (aparecen al inicio de la lista). |
| **ESC** | Salir del menú. |

---

## ModLoader e Inyecciones

El sistema de crafteo es compatible con mods mediante `get_mod_api().recipe_injections`. Esto permite añadir nuevas recetas de armas o consumibles sin modificar los archivos base del juego.

---

## Ejemplos de Uso

### Abrir la mesa de crafteo desde la ciudad

```python
from Modules.ModulesFunctions.CraftingSystem import open_crafting_menu

# Llamado al interactuar con un yunque o mesa de trabajo
open_crafting_menu(player, inventory)
```

### Consultar materiales disponibles para una receta

```python
# Lógica interna usada por el menú
def _has_materials(recipe, multiplier=1):
    for mat_id, req_qty in recipe["materials"].items():
        if inventory.get_item_quantity(mat_id) < (req_qty * multiplier):
            return False
    return True
```
