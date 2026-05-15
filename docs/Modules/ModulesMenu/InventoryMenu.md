# `InventoryMenu.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Estructura de la Interfaz](#estructura)
5. [Funciones del Módulo (API)](#funciones)
6. [Lógica de Equipamiento y Modificadores](#equipamiento)

---

## 1. Descripción General
`InventoryMenu.py` es la interfaz TUI (Text User Interface) central para que el jugador gestione y observe todos los objetos almacenados en su `vInventory`. Funciona con una vista fluida de doble panel (Izquierda: Lista con scroll inteligente; Derecha: Detalles del objeto seleccionado) actualizándose en tiempo real. Soporta 5 grandes familias de objetos: Armas (`cWeapon`), Armaduras (`cEquippableItems`), Consumibles (`cObject`), Herramientas (`cTool`), y Materiales (`cMaterial`).

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Ninguno, pero usa estructuras generadas por `setup.py` y `GatheringMasterySystem.py`.
- **Librerías TUI**: `rich.console`, `rich.panel`, `rich.table`, `rich.columns`, `rich.live`. Teclado manejado mediante `sshkeyboard`.
- **Inyecciones**: 
  - `EnchantmentSystem.py`: Llamadas perezosas (`try / except ImportError`) para interceptar las propiedades de los ítems y renderizar sus encantamientos dinámicos.
  - `EquipmentSystem.py` e `input_utils`: Delegación para cuando se aprieta "E" (Equipar).

---

## 3. Constantes y Variables Globales
- **Diccionarios Estéticos**:
  - `_RARITY_COLOR`: Colores en texto (`"Legendary": "gold1"`).
  - `_TYPE_ICON`: Emojis (`"cWeapon": "⚔"`).
  - `_TYPE_LABEL`: Strings en español (`"cWeapon": "Arma"`).
  - `_TYPE_COLOR`: Color general del panel según la familia (`"cObject": "bold yellow"`).
- `_DMG_NAMES` / `_DEF_NAMES`: Traducciones estáticas de las tuplas de resistencias (Fís, Tér, Tier, Eléc, Prof).

---

## 4. Estructura de la Interfaz

- **Panel Izquierdo (`_render_item_list`)**:
  - Lee el array de Slots de inventario (`vInventory.Objects`).
  - Muestra un máximo de ítems en pantalla. Al bajar, recalcula el `scroll_offset`.
  - Oculta el nombre con bloques `██` si `_is_discovered` devuelve False.
- **Panel Derecho (`_render_item_detail`)**:
  - Inspecciona el objeto usando `type(item).__name__` para renderizar lógicas personalizadas:
    - Armas: Imprime 5 barras `██░` de daño por elemento.
    - Armaduras: Imprime las 5 barras de defensa y extras (HP Máximo, Robo de Vida).
    - Objetos: Imprime cuanta curación o maná regeneran.
    - Herramientas: Dibuja su barra de *Efficiency* para los minijuegos de farmeo.
    - Materiales: Dibuja su *Hardness* y el skill que lo recolecta (Tala, Minería).

---

## 5. Funciones del Módulo (API)

- `open_inventory_menu(player, vInventory, console)`: Bucle principal atascante que secuestra la terminal. Inicializa el `Live` y arranca un hilo de teclado asíncrono para escuchar pulsaciones.
- `_show_item_info_screen(item, console, player)`: Sub-menú a pantalla completa. Cuando en el panel normal se aprieta la letra `[I]`, se oculta el menú dual y se pinta una tarjeta completa con toda la información y lore del objeto, esperando a que se presione `[ENTER]` para volver.
- `_choose_ring_slot(player, item, inventory, console)`: Interfaz en miniatura (desplegable) inyectada cuando se intenta equipar un anillo. Le da a elegir al usuario sobre cuál de los 3 dedos lo colocará (`ring`, `ring2`, `ring3`).

---

## 6. Lógica de Equipamiento y Modificadores

### Manejo de Modificadores (Prefijos de Rareza)
En el panel derecho, antes de mostrar el coste de Oro, el menú lee `getattr(item, "modifier", None)`. Si el objeto fue "Corrompido" o es "Legendario" (ej: *Espada Oxidada*, *Casco Titánico*), se imprime un bloque que detalla explícitamente cuáles atributos matemáticos fueron alterados (ej: `[-20% DMG, +5 Defensa]`).

### Flujo de Acción
- **`[E]` Equipar**: 
  - Solo disponible para tipos equippables.
  - Verifica la clase del objeto.
  - Si es Anillo, ejecuta `_choose_ring_slot`.
  - Pide la inyección `EquipmentManager.equip_item`.
  - El sistema actualiza pasivamente los atributos del jugador y dibuja un enorme `"✓ ACTUALMENTE EQUIPADO"` en verde debajo de la descripción.
- **`[U]` Usar**:
  - Solo para consumibles. Suma vida o maná al objeto `player` e invoca `vInventory.remove_item(item, 1)`.
- **Seguridad**: Previene equipar el mismo objeto físico (`id(objeto)`) dos veces en diferentes casillas.
