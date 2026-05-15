# `SaveSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Serialización y Deserialización](#flujo)

---

## 1. Descripción General
`SaveSystem.py` es el motor central de persistencia del estado de TPPRPG. Recoge el estado en memoria de prácticamente todos los administradores (Jugador, Inventario, Gremios, QuestSystem, Gathering, Base) y los convierte a formato JSON plano para su guardado en múltiples slots (ranuras). De manera recíproca, al cargar una partida, inyecta las variables en los módulos correspondientes asegurando que los objetos persistidos se reinstancien correctamente.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Guarda y lee desde `./Data/Saves/Slot[X].json`.
- **Inyecciones Centrales**: Este archivo es el punto de embudo de casi todo el ecosistema. Importa a:
  - `Modules.ModulesFunctions.GatheringMasterySystem`
  - `Modules.ModulesUtils.setup` (`cObject`, `cWeapon`, `cEquippableItems`)
  - `Modules.ModulesFunctions.CompanionSystem`
  - `Modules.ModulesFunctions.MasterySystem`
  - `Modules.ModulesFunctions.BaseSystem`
  - `Modules.ModulesFunctions.QuestSystem`
  - `Modules.ModulesFunctions.GuildSystem`

---

## 3. Constantes y Variables Globales
- `_SAVE_DIR` (`str`): Directorio base (`"Data/Saves"`).
- `_SLOT_PATHS` (`Dict[int, str]`): Diccionario estático que mapea IDs numéricos de Slot a su ruta real (ej: `1 -> "Data/Saves/Slot1.json"`).

---

## 4. Clases y Estructuras de Datos

*(Nota: Este módulo se basa casi enteramente en funciones puras y delegación de estado, no define clases persistentes nuevas).*

---

## 5. Funciones del Módulo (API)

### Funciones de Guardado (API Pública)
- `save_to_slot(slot: int, player, inventory, chosen_area) -> bool`
  - **Propósito**: Ejecuta toda la tubería de guardado y vuelca el resultado final en el archivo correspondiente.
  - **Retornos**: `True` si el guardado JSON tuvo éxito.

### Funciones de Carga (API Pública)
- `load_from_slot(slot: int, player, inventory, item_manager) -> Optional[dict]`
  - **Propósito**: Lee el JSON de disco. Luego, llama a todas las sub-funciones de reconstrucción (`_restore_*`) pasándole los fragmentos de diccionario relevantes.
  - **Retornos**: Retorna el bloque `game_state` si fue exitoso (que incluye variables volátiles como `chosen_area`). Retorna `None` si el slot no existe o está corrupto.
- `get_slot_info(slot: int) -> Optional[dict]`
  - **Propósito**: Lee rápidamente los metadatos superficiales del slot (Nivel del jugador, Tiempo de guardado, Oro) sin cargar ni instanciar nada. Útil para el Menú Principal.
- `delete_slot(slot: int) -> bool`
  - **Propósito**: Elimina el archivo `Slot[X].json`.

### Helpers de Serialización (`dict -> json`)
Estas funciones transforman los objetos complejos en diccionarios anidados:
- `_serialize_item(item) -> Optional[dict]`: Conserva `ID` estático y, muy importante, la lista de `enchantments` y su nivel.
- `_serialize_companion(comp) -> dict`: Guarda HP, MP, status de vida e inventario equipado de los aliados.
- `_serialize_inventory(inventory) -> list`: Itera sobre los `slots` del objeto Inventario.
- `_serialize_mastery(player) -> dict` / `_serialize_base(player) -> dict`: Llaman al método `.to_dict()` de los gestores respectivos.
- `_serialize_gathering() -> dict`: Serializa exp y niveles directamente desde la instancia global `Gathering_init`.
- `_serialize_guild_system() -> dict`: Vuelca el estado geopolítico masivo desde `GuildManager`.
- `build_save_data(player, inventory, chosen_area, active_enemies_ids) -> dict`: Función principal que consolida todos los pedazos anteriores en un mega-diccionario maestro.

### Helpers de Deserialización (`json -> RAM`)
Estas funciones instancian y llenan las clases de Python a partir de la metadata:
- `_restore_player(player, data: dict)`: Parcha el objeto `player` base sobrescribiendo atributos directos (`HP`, `Gold`, `Guild Reputations`, etc).
- `_restore_item(item_data, item_manager) -> Any`: Factoría puente. Utiliza el `item_manager` para crear el objeto base desde cero mediante su ID, y luego le reimplementa los encantos o modificadores guardados.
- `_restore_equipment(...)` / `_restore_inventory(...)`
- `_restore_companions(...)`: Llama a `load_companion_by_id` para rehidratar a los aliados.
- `_restore_mastery(...)` / `_restore_base(...)`
- `_restore_gathering(...)` / `_restore_guild_system(...)`: Rellena los Singletons con los diccionarios guardados en el archivo.

---

## 6. Flujo de Serialización y Deserialización

### Al Guardar (`save_to_slot`):
1. **Recolección**: `build_save_data` compila un diccionario con versión y timestamp.
2. **Serialización Módulos**: Llama al método `_serialize_[Modulo]` para cada subsistema.
3. **Persistencia**: Abre el archivo de manera síncrona y vuelca el diccionario estructurado.

### Al Cargar (`load_from_slot`):
1. **Lectura Base**: Carga el JSON completo.
2. **Hidratación**: Ejecuta las cadenas de `_restore_[Modulo]`. 
3. **Re-Factoría Crítica (`_restore_item`)**: Los ítems en el inventario/equipo **NO** guardan todas sus estadísticas (daño, descripciones) en el guardado. Guardan solo su `ID` y Encantamientos/Modificadores. Al cargar, el `ItemManager` vuelve a instanciar el ítem desde la base de datos `DataItems.json`, y sobre él se aplican las variaciones de la partida. Esto evita que el archivo de guardado pese megabytes y asegura compatibilidad si el desarrollador balancea el juego en parches futuros.
