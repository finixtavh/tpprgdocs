# `MainGame.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Configuración (`CONFIG`)](#configuración-config)
3. [Variables Globales del Juego](#variables-globales-del-juego)
4. [Tema Visual](#tema-visual)
5. [Funciones de Inicialización](#funciones-de-inicialización)
6. [Funciones de UI](#funciones-de-ui)
7. [Funciones de Combate](#funciones-de-combate)
8. [Funciones de Guardado/Carga](#funciones-de-guardadocarga)
9. [Bucle Principal (`main`)](#bucle-principal-main)
10. [Opciones del Menú Principal](#opciones-del-menú-principal)
11. [Ejemplos de Uso](#ejemplos-de-uso)
12. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`MainGame.py` es el **punto de entrada y bucle principal** del juego. Orquesta todos los demás módulos para presentar al jugador un RPG completo por consola con menús, combate, inventario, tiendas, compañeros, crafteo, encantamientos y base.

Al ejecutar `python MainGame.py`, se inicia el juego.

**Módulos importados principales:**

- `InventoryMenu`, `ShopManager`, `ItemManager`, `ModLoader`
- `GatheringMasterySystem`, `EnchantmentSystem`, `MasterySystem`
- `SaveSystem`, `ZoneManager`, `BuffManager`, `CraftingSystem`
- `BaseSystem`, `CompanionSystem`, `SkillTreeSystem` (vía `player.Points()`)
- `debug`, `input_utils`, `setup`

---

## Configuración (`CONFIG`)

El archivo de configuración se carga dinámicamente desde `config.json`:

```python
CONFIG = Read("config.json", None)
```

| Clave | Tipo | Descripción |
|---|---|---|
| `ASK_FOR_ADMIN` | `bool` | Si solicitar privilegios de admin al iniciar. |
| `COMBAT.DELAY_BETWEEN_ROUNDS` | `float` | Segundos de pausa entre rondas de combate. |
| `COMBAT.CHANCE_TO_FLEE` | `float` | Probabilidad base de huir (0.0–1.0). |
| `DISPLAY.BAR_LENGTH` | `int` | Longitud visual de las barras de estado. |
| `DISPLAY.SHOW_FULL_STATS` | `bool` | Si mostrar todos los stats en la pantalla de estadísticas. |
| `DEBUG.ENABLED` | `bool` | Activa el modo debug y el logging detallado. |

> **Nota:** `CONFIG` ya no es un diccionario hardcodeado en el código; se lee desde `config.json` en tiempo de ejecución. Si el archivo no existe, `Read` retornará un valor vacío y el juego fallará al acceder a las claves.

---

## Variables Globales del Juego

| Variable | Tipo | Descripción |
|---|---|---|
| `player` | `cPlayer` | Objeto del jugador. Se inicializa con `setup_player()` al importar el módulo. |
| `enemies` | `List` | Lista de enemigos disponibles. |
| `enemy` | `Optional[cEnemy]` | Enemigo actual del combate. |
| `chosen_area` | `Optional[str]` | Zona seleccionada por el jugador (`"Forest"`, `"Cave"`, `None`). |
| `shop_objects` | `List` | Items de la tienda (legado). |
| `zone_name` | `str` | Nombre de la zona actual. |
| `tick` | `int` | Contador de turnos global. |
| `chars` | `List` | Lista de personajes (reservado). |
| `item_manager` | `ItemManager` | Instancia del gestor de items. |
| `shop_manager` | `ShopManager` | Instancia del gestor de tiendas. |
| `app_logger` | `Logger` | Logger principal del juego configurado por `debug.py`. |

---

## Tema Visual

Se configura un tema de colores personalizado para `rich.Console`:

```python
custom_theme = Theme({
    "info":        "dim cyan",
    "warning":     "magenta",
    "danger":      "bold red",
    "success":     "bold green",
    "menu_title":  "bold yellow",
    "menu_option": "bright_white",
})
console = Console(theme=custom_theme)
```

Inmediatamente después se conecta el sistema de logging:

```python
app_logger = setup_logging(CONFIG["DEBUG"]["ENABLED"])
attach_console_logging(console, app_logger)
install_exception_hooks(app_logger)
```

---

## Funciones de Inicialización

### `init_game()`

Inicializa todos los sistemas del juego al arrancar:

1. Carga el `ItemManager` con `get_item_manager()`.
2. Carga el `ShopManager` con `get_shop_manager()`.
3. Inicializa y carga todos los mods con `get_mod_loader().load_all_mods()`.
4. Si existe `Data/SaveGame.json`, pregunta si cargar la partida guardada (legado, llama a `load_game()`).

---

### `load_test_items()`

Carga items de prueba (IDs 3, 4, 40, 41, 52) y los añade al inventario. Útil durante desarrollo. Acepta también `cTool` como parámetro al llamar a `load_item_by_id`.

---

### `zone_loader(player_zone: str) -> cEnemy`

Carga los enemigos de la zona indicada desde `DataStats.json` o desde los datos de mods (`mod_api.stats_injections`). Retorna un enemigo aleatorio.

**Prioridad de carga:**
1. Primero busca en `mod_api.stats_injections[player_zone]`.
2. Si no encuentra, lee de `Data/DataStats.json`.

Retorna `None` si hay error o no hay enemigos.

---

### `zone_loader_group(player_zone: str, max_enemies: int = 5) -> List[cEnemy]`

**Función nueva.** Carga un grupo de entre 1 y `max_enemies` enemigos aleatorios de la zona indicada. Sigue la misma lógica de prioridad (mods primero, luego JSON base).

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `player_zone` | `str` | Clave de la zona. |
| `max_enemies` | `int` | Máximo de enemigos en el grupo (default `5`). |

**Retorno:** Lista de instancias `cEnemy`. Lista vacía si falla la carga.

---

### `run_zone_gathering(zone_key: str) -> bool`

Ejecuta el minijuego de recolección para la zona actual. Lee los IDs de materiales del JSON, selecciona uno aleatorio, instancia el material con `item_manager.create_item_object()` y llama a `Gathering_init.gather()`. Si tiene éxito, añade el material al inventario.

Retorna `True` si se recolectó con éxito.

---

## Funciones de UI

### `display_player_status() -> None`

Muestra un panel con:
- Nombre, nivel, zona actual y oro del jugador.
- Barra de HP (con soporte de barrera `Barrier` en cian brillante si `player.Barrier > 0`).
- Barra de Maná.

---

### `create_status_bar(current, maximum, color, return_values=True)`

Crea una barra de estado visual con caracteres `#` y `-`. La longitud se toma de `CONFIG["DISPLAY"]["BAR_LENGTH"]`.

**Retorno:** Tupla `(bar_str, percentage)` si `return_values=True`. Si es `False`, imprime directamente.

---

### `create_hp_barrier_bar(health, health_max, barrier, _unused=None) -> Tuple[str, int]`

**Función nueva.** Crea una barra de HP con superposición visual de barrera (similar a los escudos de League of Legends).

**Visual:** `[CYAN_barrera | GREEN_hp | EMPTY]`

La barrera se muestra en cian brillante desde la izquierda, cubriendo las celdas de HP. La barrera es proporcional a `health_max`.

**Retorno:** Tupla `(bar_string, health_percentage)`.

```python
bar, pct = create_hp_barrier_bar(75, 100, 30)
# Barrera de 30 se renderiza sobre los primeros ~7 caracteres en cian
```

---

### `display_main_menu() -> int`

Menú principal interactivo que usa `interactive_menu_select`. Retorna el número de opción elegida.

**Opciones actuales:**

| Valor | Etiqueta |
|---|---|
| `1` | Inspeccionar las zonas |
| `2` | Ir a una zona |
| `3` | Tienda |
| `4` | Inventario Universal |
| `7` | Acciones de Zona |
| `8` | Mirar Stats |
| `9` | Gestión de Mods |
| `10` | Ver Items Disponibles |
| `11` | Guardar |
| `12` | DEBUG |
| `13` | Compañeros |
| `14` | Árbol de Habilidades |
| `15` | Mesa de Encantamientos |
| `16` | Yunque de Crafteo |
| `17` | Base |
| `0` | Salir del juego |

> **Nota:** Las opciones `5` y `6` del menú anterior (Usar objetos y Equipamiento) han sido eliminadas. El inventario unificado (opción `4`) las reemplaza.

---

### `render_combat_menu(player, enemy, combat_round)`

Muestra la UI de combate. Ahora incluye soporte para **compañeros**:

- Muestra hasta 4 paneles de compañeros activos a la izquierda del jugador.
- Compañeros caídos muestran `💀 KO` con indicación de costo de resurrección.
- El panel del jugador incluye barra de barrera si `player.Barrier > 0`.
- Las acciones disponibles son: `[1] Atacar`, `[2] Inventario`, `[3] Huir`.

---

### `display_zone_enemies(zone_name, enemies_data, strings_data)`

Muestra una tabla con los enemigos de una zona, incluyendo HP, daño físico, daño mágico total y defensa física.

---

### `get_validated_input(prompt, valid_options) -> int`

Fallback numérico simple para entradas sin menú Rich. Ignora entradas vacías (Enter residual). Atrapa `KeyboardInterrupt` y `EOFError` terminando el juego con `sys.exit(0)`.

---

### `get_zone_shop_name(zone_key: str) -> str`

Retorna el nombre de la tienda de una zona: `f"{zone_key}Shop"`.

---

## Funciones de Combate

### `apply_debuffs(player) -> Dict`

Aplica todos los debuffs activos en `player.applieddebuffs`. Para cada debuff, llama a `debuff.Apply(attr, player)` por cada atributo no-callable. Retorna un diccionario de efectos aplicados.

---

## Funciones de Guardado/Carga

### `save_game() -> bool`

Guarda el estado básico del juego en `Data/SaveGame.json` (función legado, preservada para compatibilidad).

**Datos guardados:**

```json
{
    "player": { "name": "...", "level": 1, "health": 100, ... },
    "game_state": { "area": "Forest", "tick": 0 },
    "timestamp": "..."
}
```

> **Nota:** El guardado completo (inventario, equipamiento, compañeros, masteries, base) lo gestiona `SaveSystem.py` a través de la opción `11` del menú.

---

### `load_game() -> bool`

Carga el archivo `Data/SaveGame.json` (legado) y restaura los atributos básicos del jugador. Retorna `False` si el archivo no existe o está corrupto.

---

## Bucle Principal (`main`)

La función `main()` se llama en un loop infinito desde el bloque `if __name__ == "__main__"`. En cada iteración:

1. Llama a `player.Check(vInventory)` y regenera el maná.
2. Limpia la pantalla.
3. Muestra el estado del jugador y el menú principal.
4. Ejecuta la acción correspondiente.

El loop externo captura excepciones y las muestra en consola sin terminar el juego.

---

## Opciones del Menú Principal

| Opción | Acción |
|---|---|
| `0` | Salir del juego (con confirmación `s/n`). |
| `1` | Ver información de todas las zonas dinámicamente (usa `get_all_zones()`). Muestra fuente si es de un mod. |
| `2` | Ir a una zona (selector dinámico con `get_zones_for_selector()`). Dispara evento `zone_entered` para mods. |
| `3` | Tienda general con submenú: GeneralStore, WeaponShop, ArmorShop, MagicShop. |
| `4` | Inventario universal (`open_inventory_menu`). |
| `7` | Acciones de zona: Luchar, Tienda de zona, Recolectar, Volver. |
| `8` | Ver estadísticas completas del jugador con barras, defensas multi-tipo, compañeros, buffs activos y equipamiento. |
| `9` | Gestión de mods (listar, recargar, ver estado). |
| `10` | Ver todos los items disponibles en el juego. |
| `11` | Guardar/Cargar partida mediante `save_load_menu` de `SaveSystem.py` (3 slots). |
| `12` | Menú DEBUG (añadir armas, oro/EXP, mastery, variables, compañero de prueba). |
| `13` | Gestión de compañeros (`companion_management_menu`). |
| `14` | Árbol de Habilidades (`player.Points()`). |
| `15` | Mesa de Encantamientos (`open_enchantment_menu`). |
| `16` | Yunque de Crafteo (`open_crafting_menu`). |
| `17` | Base del jugador (`open_base_menu`), con inyección de edificios de mods. |

---

### Flujo de Zona (opción 7)

```
¿Hay zona seleccionada?
    NO → Mensaje de error
    SÍ → Submenú:
        [1] Luchar
            - Carga enemigo si no hay uno con zone_loader()
            - Bucle de combate por rondas:
                · Turno del jugador: Atacar / Inventario / Huir
                · Turno de compañeros (con acciones según clase)
                · Turno del enemigo (ataca a objetivo aleatorio: jugador o compañero)
            - Al vencer: EXP, oro, loot aleatorio, posible level-up
        [2] Tienda actual de la zona
            - Busca tienda "{ZoneKey}Shop"
            - Si existe, abre display_shop()
        [3] Recolectar
            - Llama a run_zone_gathering(chosen_area)
        [4] Devolverse al menú principal
```

### Flujo de combate con compañeros (detalle)

Durante el turno de cada compañero vivo, el jugador elige su acción según la clase del compañero:

| Clase | Acciones disponibles |
|---|---|
| Cualquiera | `[1] Atacar` |
| `Curandero` | `[2] Curar al jugador (+20% HP máx)`, `[3] Curar al equipo (+15% HP máx)` |
| `Soporte` | `[2] Debilitar al enemigo (debuff vía BuffManager o reducción DEF -10%)` |

El enemigo elige un objetivo aleatorio entre el jugador y los compañeros vivos.

---

### Menú DEBUG (opción 12)

| Sub-opción | Acción |
|---|---|
| `1` | Añadir arma de prueba (IDs 3 y 100) al inventario. |
| `2` | Añadir 9.999.999 de oro y EXP. |
| `3` | Subir 10 niveles a todas las skills de Gathering y Armas. |
| `4` | Mostrar `dump_debug_snapshot` en pantalla y loggear. |
| `5` | Añadir compañero de prueba `Lyra` con arma equipada si hay una en el inventario. |

---

## Ejemplos de Uso

### Ejecutar el juego

```bash
python MainGame.py
```

### Cargar un grupo de enemigos para un encuentro

```python
group = zone_loader_group("Forest", max_enemies=3)
# Retorna lista de 1 a 3 instancias de cEnemy del Bosque
```

### Crear barra con barrera

```python
bar, pct = create_hp_barrier_bar(60, 100, 25)
# Bar: [CYAN×6 | GREEN×9 | EMPTY×10]
# pct: 60
```

---

## Notas y Referencias

- **`rich` library:** Usada extensivamente para UI. [https://rich.readthedocs.io/](https://rich.readthedocs.io/)
- **`Enum` en Python:** Para estados del juego. [https://docs.python.org/3/library/enum.html](https://docs.python.org/3/library/enum.html)
- El `CONFIG` ahora se lee desde `config.json` en lugar de estar hardcodeado. Si el archivo no existe el juego no arrancará correctamente.
- El **guardado completo** (con inventario, masteries, compañeros y base) está en `SaveSystem.py`. La función `save_game()` en este archivo es un guardado básico de legado.
- La probabilidad de huir está limitada entre 5% y 95%: `flee_chance = max(0.05, min(0.95, flee_chance))`.
- El evento `zone_entered` se dispara automáticamente al seleccionar una zona, permitiendo a los mods reaccionar a la entrada.
- La opción `14` del menú llama a `player.Points()`, que abre el árbol de habilidades definido en `SkillTreeSystem.py`.
