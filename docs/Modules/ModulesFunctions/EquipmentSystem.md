# `EquipmentSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Equipar y Desequipar](#flujo-de-equipamiento)

---

## 1. Descripción General
`EquipmentSystem.py` es el módulo responsable de manejar la interfaz de usuario y la lógica matemática para equipar y desequipar ítems del personaje (Armas, Armaduras, Cascos, Botas, Anillos y Herramientas). Se encarga de extraer las estadísticas pasivas de los objetos (Defensa, Vida Extra, Robo de Vida, etc.) y aplicarlas de manera reversible al jugador, asegurando que cuando un ítem es desequipado, sus bonificaciones se restan sin causar bugs de pérdida de estadísticas.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Ninguno, no lee JSON directamente, lee atributos en tiempo de ejecución.
- **Inyecciones**:
  - `ModulesUtils.input_utils` (`interactive_menu_select`, `wait_for_enter`) para la renderización de la UI.
  - Clases estructurales `cTool` (desde `GatheringMasterySystem.py`) y `cWeapon` (desde `setup.py`) cargadas mediante importaciones perezosas (`lazy imports`) para evitar ciclos cruzados y reconocer correctamente su tipo mediante `isinstance`.

---

## 3. Constantes y Variables Globales
- No declara variables globales sueltas. Todo está encapsulado en métodos estáticos dentro de la clase administradora.

---

## 4. Clases y Estructuras de Datos

### `EquipmentManager`
Factoría 100% estática (`@staticmethod`). No necesita ser instanciada.

#### Constantes de Clase
- `VALID_SLOTS` (`Dict`): Un mapeo entre el atributo `objectcheck` que tienen los ítems y el atributo de instancia que posee el jugador. 
  ```python
  {
      "Weapon": "weapon",
      "Helmet": "helmet", 
      "Armor": "armor",
      "Boots": "boots",
      "Ring": ["ring", "ring2", "ring3"] # Soporta 3 slots de anillo.
  }
  ```

#### Métodos de Matemática Pura
- `get_item_stats(item) -> dict`: Escanea el objeto pasado por parámetro buscando atributos como `Defense`, `HealthBoost`, `DmgBoost` y `HealthSteal`. Empaqueta lo que encuentre en un diccionario estandarizado. *(Las armas no aplican daño base aquí, ya que el motor de combate extrae su daño dinámicamente).*
- `apply_item_stats(player, item)`: Desempaqueta el diccionario de `get_item_stats`. Para las Defensas (`DEF`), recorre el array sumando resistencias específicas. Para vida/daño, usa `setattr`. Al finalizar marca el ítem con `item.AppliedStats = True`.
- `remove_item_stats(player, item)`: Operación recíproca. Si el ítem estaba marcado como aplicado, resta sus estadísticas del jugador para dejarlo limpio.

#### Métodos de Control
- `find_available_ring_slot(player) -> Optional[str]`: Itera los slots `"ring"`, `"ring2"`, `"ring3"` buscando el primero que tenga `None`.
- `equip_item(player, item, inventory=None) -> bool`: Controlador de cambio. Revisa el slot requerido, invoca a `remove_item_stats` si ya había un ítem en esa casilla, lo manda al inventario provisto, coloca el nuevo y llama a `apply_item_stats`. 
- `unequip_item(player, slot_name, inventory=None) -> Optional[object]`: Fuerza el vaciado de un slot y el recálculo matemático, depositando el ítem en la mochila.
- `show_equipment(player)`: Imprime en consola ASCII un resumen detallado de las partes corporales y qué lleva equipado, incluyendo rangos de daño y tipos de defensa elemental de cada prenda.

---

## 5. Funciones del Módulo (API)

- `equip_menu(player, inventory)`
  - **Propósito**: Ejecuta un bucle `while True` que muestra la interfaz visual de interacción con el jugador. Permite seleccionar 1. Equipar, 2. Desequipar, 3. Ver Equipo.
  - **Interacción**: Cuando el jugador escoge "Equipar", el script filtra el inventario utilizando `hasattr(item, 'objectcheck')` o revisando si es `cTool/cWeapon` para mostrar solo cosas ponibles. Invoca a `interactive_menu_select` para elegir y orquesta los movimientos.

---

## 6. Flujo de Equipar y Desequipar

### Al Equipar un Anillo Nuevo en un inventario con 1 Anillo ya puesto:
1. Jugador abre `equip_menu` y escoge un `Anillo` de su mochila.
2. `equip_item` detecta que su `objectcheck == "Ring"`.
3. Inicia un menú especial interactivo listando los 3 slots de anillo y qué hay en ellos.
4. Si el jugador elige el "Slot 2 (Vacío)", la función asigna `slot_name = "ring2"`.
5. Verifica `getattr(player, "ring2")`. Es `None`. No hace falta desequipar nada.
6. Llama `setattr(player, "ring2", Anillo_Nuevo)`.
7. `apply_item_stats` busca el `HealthBoost` del anillo y lo suma al jugador temporalmente marcando `AppliedStats = True`.
8. El sistema descuenta 1 unidad de este Anillo del inventario `inventory.remove_item(item, 1)`.

### Consideraciones sobre Herramientas (`cTool`) y Armas (`cWeapon`)
Estos dos tipos de objetos son heredados de sus propios subsistemas y **carecen** del atributo de compatibilidad universal `objectcheck`. El gestor contiene parches rígidos `isinstance(item, cTool)` y `isinstance(item, cWeapon)` a mitad del método para esquivar esta regla y enrutarlos forzosamente hacia `player.tool` y `player.weapon` respectivamente.
