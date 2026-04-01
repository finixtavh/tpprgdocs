# `InventoryMenu.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Constantes Visuales](#constantes-visuales)
3. [Funciones Helper Internas](#funciones-helper-internas)
4. [Paneles de Renderizado](#paneles-de-renderizado)
5. [Pantalla de Información Detallada](#pantalla-de-información-detallada)
6. [Función Principal `open_inventory_menu`](#función-principal-open_inventory_menu)
7. [Ejemplos de Uso](#ejemplos-de-uso)
8. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`InventoryMenu.py` implementa el **menú de inventario interactivo** del juego. Muestra un layout de dos columnas —lista de items a la izquierda, panel de detalle a la derecha— con navegación por flechas en tiempo real usando `rich.Live`.

Es compatible con todos los tipos de item del juego:

| Tipo de item | Acciones disponibles |
|---|---|
| `cWeapon` | `[E] Equipar`, `[I] Información` |
| `cEquippableItems` | `[E] Equipar`, `[I] Información` |
| `cTool` | `[E] Equipar`, `[I] Información` |
| `cObject` | `[U] Usar`, `[I] Información` |
| `cMaterial` | `[I] Información` |

---

## Constantes Visuales

### `_RARITY_COLOR`

```python
_RARITY_COLOR = {
    "Common":    "white",
    "Uncommon":  "bright_green",
    "Rare":      "bright_blue",
    "Epic":      "medium_purple1",
    "Legendary": "gold1",
}
```

Mapea la rareza de un material a su color Rich para el panel de detalle.

---

### `_TYPE_ICON`, `_TYPE_LABEL`, `_TYPE_COLOR`

Tres diccionarios que mapean el nombre de clase del item (`cWeapon`, `cEquippableItems`, etc.) a su icono emoji, etiqueta de texto y color de estilo Rich respectivamente:

| Tipo | Icono | Etiqueta | Color |
|---|---|---|---|
| `cWeapon` | `⚔` | `Arma` | `bold red` |
| `cEquippableItems` | `🛡` | `Equipamiento` | `bold cyan` |
| `cObject` | `🧪` | `Objeto` | `bold yellow` |
| `cTool` | `⛏` | `Herramienta` | `bold dark_orange3` |
| `cMaterial` | `🪨` | `Material` | `bold blue` |

---

### `_DMG_NAMES` / `_DEF_NAMES`

```python
_DMG_NAMES = ["Fís", "Tér", "Tier", "Eléc", "Prof"]
_DEF_NAMES = ["Fís", "Tér", "Tier", "Eléc", "Prof"]
```

Etiquetas abreviadas para los 5 tipos de daño/defensa, usadas en los paneles de detalle.

---

## Funciones Helper Internas

### `_item_type(item) -> str`
Retorna `type(item).__name__` (ej: `"cWeapon"`, `"cObject"`).

### `_item_name(item) -> str`
Busca el nombre del item probando los atributos `Name`, `name` en ese orden. Retorna `"???"` si ninguno existe.

### `_item_desc(item) -> str`
Busca la descripción probando `Description`, `description`, `Desc`, `desc`. Retorna `"Sin descripción."` si ninguno existe.

### `_item_cost(item) -> int`
Retorna `Gold_Cost` del item o `0` si no tiene.

---

## Paneles de Renderizado

### `_render_item_list(slots, selected_idx, scroll_offset, max_visible) -> Panel`

Genera el panel izquierdo con la lista paginada del inventario.

**Características:**
- Muestra hasta `max_visible` slots a la vez (por defecto `14` en `open_inventory_menu`).
- La fila seleccionada se resalta con **fondo blanco brillante y texto negro** (`black on bright_white`).
- El cursor de selección muestra `►` en la fila activa.
- Un footer indica el rango visible: `"1–14 de 32  [↑/↓ para mover]"`.
- Cada fila muestra: índice, icono de tipo, nombre, etiqueta de tipo y cantidad (si > 1).

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `slots` | `List[InventorySlot]` | Lista de slots del inventario. |
| `selected_idx` | `int` | Índice absoluto del slot seleccionado. |
| `scroll_offset` | `int` | Primer slot visible (para paginación). |
| `max_visible` | `int` | Número máximo de slots visibles simultáneamente. |

---

### `_render_item_detail(slot, player=None) -> Panel`

Genera el panel derecho con los detalles del item seleccionado.

**Contenido según tipo:**

| Tipo | Información mostrada |
|---|---|
| `cWeapon` | Barras visuales de daño por tipo (mínimo–máximo), con barra de `█░`. El tipo Profundo siempre muestra `(fijo)`. |
| `cEquippableItems` | Barras de defensa por tipo, HP máximo (`❤`), robo de vida (`🩸`), daño extra (`⚔`), slot de equipamiento. |
| `cObject` | HP restaurado (`❤`), MP restaurado (`💧`). |
| `cTool` | Barra de eficiencia, tipo de material compatible, encantamientos. |
| `cMaterial` | Rareza (con color), barra de dureza, skill requerida, tipos de material. |

**Indicador de equipado:** Si `player` es proporcionado y el item está actualmente equipado, muestra `"✓ ACTUALMENTE EQUIPADO"` en verde brillante.

Si `slot` es `None`, muestra un panel vacío con mensaje de ayuda.

---

### `_render_actions(slot, message, message_style) -> Panel`

Genera el panel inferior con las acciones disponibles para el item seleccionado.

- Para `cWeapon`, `cEquippableItems`, `cTool`: muestra `[E] Equipar  [I] Información  [Q] Cerrar`.
- Para `cObject` y otros: muestra `[U] Usar  [I] Información  [Q] Cerrar`.
- Si hay un `message` activo (ej: confirmación de éxito o error), lo muestra debajo de las acciones con el estilo `message_style`.

---

## Pantalla de Información Detallada

### `_show_item_info_screen(item, console)`

Muestra una pantalla completa (a pantalla completa, `console.clear()`) con toda la información del item seleccionado. Espera cualquier pulsación de tecla para volver al menú principal de inventario.

**Información mostrada según tipo:**

- **`cWeapon`:** Tabla de daño mínimo–máximo por tipo.
- **`cEquippableItems`:** Slot, tabla de defensa, HP, robo de vida, daño extra.
- **`cObject`:** Efectos al usar (HP y MP restaurados).
- **`cTool`:** Eficiencia, tipo de material, encantamientos.

---

## Función Principal `open_inventory_menu`

```python
open_inventory_menu(inventory, player=None, *, on_equip=None, on_use=None) -> None
```

Abre el menú de inventario interactivo y entra en el bucle de eventos. Bloquea hasta que el usuario cierra el menú con `[Q]` o `Escape`.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `inventory` | `cInventory` | Inventario del jugador. Debe tener atributo `slots` (lista de `InventorySlot`). |
| `player` | `cPlayer \| None` | Objeto del jugador. Si se proporciona, se muestra el indicador de "equipado" y se pueden aplicar efectos de items. |
| `on_equip` | `Callable(player, item, inventory) → bool \| None` | Callback para equipar un item. Si es `None`, usa `EquipmentManager.equip_item` por defecto. |
| `on_use` | `Callable(player, item, inventory) → bool \| None` | Callback para usar un `cObject`. Si es `None`, aplica `HealthRestore` y `ManaRestore` directamente. |

### Controles del menú

| Tecla | Acción |
|---|---|
| `↑` | Seleccionar item anterior. |
| `↓` | Seleccionar item siguiente. |
| `E` | Equipar item seleccionado (`cWeapon`, `cEquippableItems`, `cTool`). |
| `U` | Usar item seleccionado (`cObject`). |
| `I` | Abrir pantalla de información detallada. |
| `Q` / `Escape` | Cerrar el menú de inventario. |

### Comportamiento al equipar (`E`)

1. Llama a `on_equip(player, item, inventory)`.
2. Si retorna `True`: elimina el item del inventario con `inventory.remove_item(item, 1)` y muestra mensaje de éxito.
3. Si retorna `False`: muestra mensaje de error.

### Comportamiento al usar (`U`)

**Con `on_use` definido:**
- Llama a `on_use(player, item, inventory)`.
- Si retorna `True`, elimina el item del inventario.

**Sin `on_use` (comportamiento por defecto):**
- Lee `HealthRestore` y `ManaRestore` del item.
- Aplica la curación respetando `Health_max` y `Mana_max`.
- Elimina el item del inventario.
- Muestra el HP y MP curados en el mensaje de resultado.

### Implementación técnica

- Usa `rich.Live` con 30 fps (`refresh_per_second=30`) y `transient=True`.
- El listener de teclado corre en un hilo daemon separado.
- El bucle exterior (`while True`) permite volver al menú tras acciones como Info o Equipar sin tener que reinicializar todo el estado.
- Usa `keyboard_listener_scope()` de `input_utils.py` para serializar el listener.

### Scroll automático

El índice seleccionado siempre permanece visible. Si sube del límite superior o baja del inferior de la ventana visible, `_clamp()` ajusta `scroll_offset` para seguir al cursor.

---

## Ejemplos de Uso

### Abrir el inventario con comportamiento por defecto

```python
from Modules.InventoryMenu import open_inventory_menu
from Modules.setup import vInventory

# Sin jugador (solo lectura/información)
open_inventory_menu(vInventory)

# Con jugador (permite equipar y usar)
open_inventory_menu(vInventory, player=player)
```

### Con callbacks personalizados

```python
from Modules.InventoryMenu import open_inventory_menu

def mi_equip(player, item, inventory):
    """Lógica personalizada de equipamiento."""
    # ...
    return True  # True = éxito

def mi_use(player, item, inventory):
    """Lógica personalizada al usar consumible."""
    # ...
    return True

open_inventory_menu(vInventory, player=player, on_equip=mi_equip, on_use=mi_use)
```

### Integración en el bucle de MainGame

```python
# En MainGame.py, opción de menú para inventario:
from Modules.InventoryMenu import open_inventory_menu

open_inventory_menu(vInventory, player=player)
```

---

## Notas y Referencias

- El menú requiere que `cInventory` exponga un atributo `slots` (lista de `InventorySlot`). La clase `cInventory` de `setup.py` lo cumple.
- Si el inventario está vacío, todos los paneles se muestran en estado "vacío" sin errores.
- **`rich.Live`:** [https://rich.readthedocs.io/en/stable/live.html](https://rich.readthedocs.io/en/stable/live.html)
- **`rich.Columns`:** Permite mostrar varios renderizables en columnas. [https://rich.readthedocs.io/en/stable/columns.html](https://rich.readthedocs.io/en/stable/columns.html)
- **`sshkeyboard`:** Librería de captura de teclado usada para la navegación. [https://github.com/sagi-z/sshkeyboard](https://github.com/sagi-z/sshkeyboard)
