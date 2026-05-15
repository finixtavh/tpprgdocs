# `EnchantmentSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Catálogo y Mutación de Niveles](#mecanica)

---

## 1. Descripción General
`EnchantmentSystem.py` permite la personalización mágica del equipamiento. Añade modificadores matemáticos permanentes a `cWeapon`, `cEquippableItems` y `cTool` mediante un sistema de Familias y Niveles Romanos. Si un jugador aplica un encantamiento dos veces a la misma pieza de equipo, este sube de nivel (I → II → III) mejorando sus bonificaciones. Además, proporciona una interfaz de consola (Mesa de Encantamientos) donde el jugador puede gastar oro interactuando directamente con sus objetos.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Carga automáticamente el catálogo completo leyendo `./Data/DataEnchantments.json` al arrancar.
- **Inyecciones**:
  - Parchea las importaciones condicionales y comprueba clases ajenas usando `isinstance` importando temporalmente `cTool` y `cWeapon`.
  - Exporta métodos directos para que `CombatEngine` y `GatheringMasterySystem` extraigan pasivamente los efectos durante cálculos matemáticos.

---

## 3. Constantes y Variables Globales
- `MAX_ENCHANT_SLOTS` (`int`): Límite físico de familias únicas por ítem (por defecto `5`). Un ítem puede tener `Vampirismo II` y `Fuego I`, ocupando 2 de 5 slots.
- `_ROMAN` (`Dict`): Diccionario para la conversión visual de nivel entero a formato romano.
- `ENCHANTMENTS_CATALOG` (`Dict[str, cEnchantment]`): Caché en memoria ram.
- `_BASE_LOADED` (`bool`): Hook de auto-carga (`True` tras la primera importación).

---

## 4. Clases y Estructuras de Datos

### `cEnchantment`
Contenedor lógico extraído del JSON. Representa la *plantilla matemática* del encantamiento, no su aplicación.

#### Atributos de Instancia (Relevantes)
- `self.TargetClass` (`str`): Qué tipo de objetos admite (`cWeapon`, `cEquippableItems`, etc).
- `self.Effects` (`Dict[str, Any]`): Diccionario clave-valor con la matemática bruta (`"lifesteal": 0.15`).
- `self.MaxLevel` (`int`): Nivel tope del encantamiento.
- `self.family_id` (`str`): String interno que agrupa los niveles. (ej. `"sharpness"`).
- `self.max_level_name` (`str`): String opcional. Si un encantamiento llega al tope, puede mutar su nombre cosmético en la interfaz.

#### Filtros y Parseo
- `can_apply_to(self, item) -> bool`: Función inteligente que además de revisar `TargetClass`, normaliza y quita acentos del `item.Name` para aceptar "Espada" si el target es "espada".
- `apply_to_weapon()`, `apply_to_equipment()`, `apply_to_tool()`: Extrapolan los datos de `self.Effects` y los formatean en un diccionario compatible con las máquinas de estado finales del jugador. Ej: Transforma `dmg_bonus: 0.10` a `{"dmg_multiplier": 1.10}`.

---

## 5. Funciones del Módulo (API)

### Auto-Mantenimiento
- `load_base_enchantments()`: Abre el JSON y alimenta el `ENCHANTMENTS_CATALOG`.

### Interfaz del Objeto
- `get_item_enchants(item) -> List[Dict]`: Extrae la propiedad `Enchantments` del objeto de inventario. Contiene lógica legacy de migración para convertir listas antiguas `["Fire", "Ice"]` al nuevo formato con diccionario `{"family_id": "fire", "name": "Fire", "level": 1}`.
- `count_unique_families(item) -> int`: Retorna cuántos de los `MAX_ENCHANT_SLOTS` están ocupados.

### Consola Interactiva
- `open_enchantment_menu(player, vInventory)`: Interfaz en consola `rich`.
  1. Busca referencias en memoria de todos los slots equipados (Armor, SecHand, Ring3...).
  2. Itera el Inventario buscando equipo no vestido.
  3. Muestra una lista al jugador.
  4. Al seleccionar un objeto, muestra el Submenú del Catálogo filtrando (`get_enchantments_for_item`) solo los compatibles.

---

## 6. Catálogo y Mutación de Niveles

1. El Jugador tiene una espada con `Sharpness I` (Coste: 100 Oro).
2. Entra a la Mesa de Encantamientos.
3. El motor revisa que la familia `"sharpness"` ya existe en la lista de ese objeto.
4. Le permite "Actualizar". El costo se multiplica: $CosteBase \times (NivelActual + 1) = 200 Oro$.
5. Acepta. El nivel se sobrescribe en la mochila a `2`.
6. En pantalla, gracias a `format_enchant_display()`, ahora la espada reza `Sharpness II`.
7. Si llegase al nivel final (por ejemplo 5) y el JSON tenía configurado `"max_level_name": "Destrucción"`, el texto cambia automáticamente la próxima vez que se renderice.
