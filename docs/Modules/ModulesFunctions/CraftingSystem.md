# `CraftingSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Crafteo y Reforjado](#flujo)

---

## 1. Descripción General
`CraftingSystem.py` maneja la creación de objetos a partir de materias primas recolectadas por el jugador. Su principal función es exponer una interfaz de usuario pesada que dibuja un menú dinámico en consola (`rich`) donde el jugador puede ver recetas, materiales faltantes, buscar por nombre, y, como adición avanzada, alternar a un "Modo Reforjado" para gastar oro modificando proceduralmente la rareza de los ítems en su inventario.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere el JSON `./Data/DataRecipes.json` donde se declaran las fórmulas base.
- **Inyecciones Condicionales**: 
  - `ModulesUtils.Loader` (lectura).
  - `ModulesUtils.ModLoader` (las recetas pueden ser inyectadas externamente).
  - `ItemManager` (inyección global para la instanciación factoría).
  - `setup.py` (importación de clases hijas como `cWeapon` para inyectarlas a la factoría).
  - `ItemModifierSystem` (inyectado *in-place* al finalizar el crafteo o durante un re-roll).
- **Librerías TUI**: `rich` (Live, Table, Panel) y `sshkeyboard`.

---

## 3. Constantes y Variables Globales
- **No se exportan variables globales en este módulo.** El objeto principal `console = Console()` es del ámbito local de UI.
- En la función principal, la constante `REFORGE_COST = 500` determina el costo de oro de la mecánica secundaria.

---

## 4. Clases y Estructuras de Datos

### `CraftingSystem`
Motor transitorio de carga de recetas.

#### Atributos de Instancia
- `self.player`: Puntero al objeto del jugador actual.
- `self.inventory`: Puntero al inventario del jugador.
- `self.recipes` (`List[Dict]`): Lista normalizada de recetas. Cada receta tiene el formato:
  ```json
  {
      "id": "espada_hierro",
      "result_name": "Espada de Hierro",
      "result_id": 5,
      "result_qty": 1,
      "materials": {"iron_ingot": 2, "wood": 1},
      "gold_cost": 0
  }
  ```

#### Métodos Principales
- `__init__(self, player, inventory)`: Guarda referencias y llama a `_load_recipes()`.
- `_load_recipes(self)`: Parsea `DataRecipes.json`. Normaliza los nombres de los atributos. Adicionalmente, verifica `mod_api.recipe_injections` por si algún mod añadió nuevas recetas.

---

## 5. Funciones del Módulo (API)

- `open_crafting_menu(player, inventory)`
  - **Propósito**: Bucle principal asíncrono y síncrono mixto del menú de creación. Utiliza `rich.live` para actualizar la pantalla a 15fps.
  - **Parámetros**: `player` (clase jugador), `inventory` (gestor de inventario).
  - **Estructura Interna**: 
    Contiene subclausuras (`closures`) fuertemente acopladas al estado local (`state = {"idx": 0, "input_mode": "normal", ...}`).
    - `_get_actual_item_id(mat_id_or_name)` y `_get_actual_item_name()`: Resuelven identificadores mágicos utilizando el Singleton del `ItemManager`.
    - `_has_materials(recipe, multiplier=1)`: Check de viabilidad de recursos en tiempo real.
    - `_craft(recipe, qty=1)`: Elimina el costo del inventario y el oro, manda a instanciar el ítem resultante, invoca al `ItemModifierSystem` pasándole los multiplicadores de calidad `GLOBAL_CRAFTING_QUALITY_MODIFIER`, y lo añade al inventario. Además añade el ID a la lista de `discovered_items` del jugador.
    - `_get_forgeable_slots()`: Filtra el inventario a solo Armas, Equipables y Herramientas (útil para el menú secundario de Reforja).
    - `_do_reforge()`: Función secundaria que descuenta 500 de Oro, llama a `remove_current_modifier()` para limpiar el ítem, y aplica uno nuevo a través de `roll_and_apply_modifier`.
  - **Controles (Listeners)**:
    - Modo Normal: Flechas para navegar la lista.
    - `[S]`: Cambia el modo para escribir strings de búsqueda.
    - `[F]`: Alterna favoritos (sube la receta al principio de la lista de dibujado).
    - `[R]`: Cambia totalmente la pantalla de `Crafting` a `Reforge`.
    - `[Enter]`: Envía un comando de crafteo o de reforjado según la pestaña activa.

---

## 6. Flujo de Crafteo y Reforjado

### Ciclo Normal
1. Jugador entra a Crafteo.
2. Interfaz llama `_has_materials()`, si es positivo habilita color Verde.
3. Jugador pulsa `Enter`.
4. El Inventario pierde los recursos, la Factoría genera el Ítem en memoria.
5. `ItemModifierSystem.py` tira un "Roll" de suerte sobre el objeto.
6. El objeto resultante es un `Ítem [Modificado]` y aterriza en el inventario final.

### Ciclo de Reforja
1. Jugador presiona `R` en Crafteo.
2. Interfaz lista solo su armadura/armas actuales.
3. Jugador selecciona objeto y paga oro.
4. El sistema limpia sus stats pasadas, restaura la vida/daño original, y ejecuta un "Roll" nuevo de `ItemModifierSystem.py`.
5. Si falla el roll sale malo, si lo gana sale excelente. El ítem se actualiza en tiempo real en la mochila.
