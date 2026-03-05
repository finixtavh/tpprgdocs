# `MainGame.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Configuración (`CONFIG`)](#configuración-config)
3. [Estado del Juego (`GameState`)](#estado-del-juego-gamestate)
4. [Variables Globales del Juego](#variables-globales-del-juego)
5. [Tema Visual](#tema-visual)
6. [Funciones de Inicialización](#funciones-de-inicialización)
7. [Funciones de UI](#funciones-de-ui)
8. [Funciones de Combate](#funciones-de-combate)
9. [Funciones de Guardado/Carga](#funciones-de-guardadocarga)
10. [Bucle Principal (`main`)](#bucle-principal-main)
11. [Opciones del Menú Principal](#opciones-del-menú-principal)
12. [Ejemplos de Uso](#ejemplos-de-uso)
13. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`MainGame.py` es el **punto de entrada y bucle principal** del juego. Orquesta todos los demás módulos (Loader, setup, EquipmentSystem, ItemManager, ShopManager, ModLoader, GatheringMasterySystem) para presentar al jugador un RPG completo por consola con menús, combate, inventario y tiendas.

Al ejecutar `python MainGame.py` (o `python -m Modules.MainGame`), se inicia el juego.

---

## Configuración (`CONFIG`)

Diccionario global con todas las opciones configurables del juego:

```python
CONFIG = {
    "ASK_FOR_ADMIN": False if platform.platform == "Windows" else True,

    "COMBAT": {
        "DELAY_BETWEEN_ROUNDS": 3.5,   # Segundos entre rondas
        "CHANCE_TO_FLEE": 0.3,          # Probabilidad base de huir (30%)
    },
    "DISPLAY": {
        "BAR_LENGTH": 25,               # Largo de las barras HP/MP
        "MENU_REFRESH_RATE": 0.5,       # Segundos de espera al refrescar menús
        "SHOW_FULL_STATS": True,        # Mostrar stats completos en pantalla
    },
    "DEBUG": {
        "ENABLED": False,               # Modo debug (actualmente no hace nada)
    }
}
```

| Clave | Tipo | Descripción |
|---|---|---|
| `ASK_FOR_ADMIN` | `bool` | Si solicitar privilegios de admin al iniciar. |
| `COMBAT.DELAY_BETWEEN_ROUNDS` | `float` | Segundos de pausa entre rondas de combate. |
| `COMBAT.CHANCE_TO_FLEE` | `float` | Probabilidad base de huir (0.0–1.0). |
| `DISPLAY.BAR_LENGTH` | `int` | Longitud visual de las barras de estado. |
| `DISPLAY.SHOW_FULL_STATS` | `bool` | Si mostrar todos los stats en la pantalla de estadísticas. |
| `DEBUG.ENABLED` | `bool` | Modo debug (reservado). |

---

## Estado del Juego (`GameState`)

Enum que registra en qué pantalla se encuentra el juego actualmente:

```python
class GameState(Enum):
    MAIN_MENU  = auto()
    EXPLORING  = auto()
    COMBAT     = auto()
    SHOP       = auto()
    INVENTORY  = auto()
    STATS      = auto()
    EQUIPMENT  = auto()
```

---

## Variables Globales del Juego

| Variable | Tipo | Descripción |
|---|---|---|
| `player` | `cPlayer` | Objeto del jugador. Se inicializa con `setup_player()`. |
| `enemies` | `List` | Lista de enemigos disponibles. |
| `enemy` | `Optional[cEnemy]` | Enemigo actual del combate. |
| `chosen_area` | `Optional[str]` | Zona seleccionada por el jugador (`"Forest"`, `"Cave"`, `None`). |
| `shop_objects` | `List` | Items de la tienda (legado). |
| `zone_name` | `str` | Nombre de la zona actual. |
| `tick` | `int` | Contador de turnos global. |
| `chars` | `List` | Lista de personajes (reservado). |
| `current_state` | `GameState` | Estado actual del juego. |
| `item_manager` | `ItemManager` | Instancia del gestor de items. |
| `shop_manager` | `ShopManager` | Instancia del gestor de tiendas. |

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

Uso en el código:

```python
console.print("[success]¡Éxito![/success]")
console.print("[danger]¡Error crítico![/danger]")
```

---

## Funciones de Inicialización

### `init_game()`

Inicializa todos los sistemas del juego al arrancar:

1. Carga el `ItemManager` con `get_item_manager()`.
2. Carga el `ShopManager` con `get_shop_manager()`.
3. Inicializa y carga todos los mods con `get_mod_loader().load_all_mods()`.
4. Si existe `Data/SaveGame.json`, pregunta si cargar la partida guardada.

---

### `load_test_items()`

Carga algunos items de prueba (IDs 3, 4, 40, 41, 52) y los añade al inventario. Útil durante desarrollo.

---

### `zone_loader(player_zone: str) -> cEnemy`

Carga los enemigos de la zona indicada desde `DataStats.json` y retorna uno aleatorio.

| Parámetro | Descripción |
|---|---|
| `player_zone` | `"Forest"` o `"Cave"` |

Retorna una instancia de `cEnemy` o `None` si hay error.

---

## Funciones de UI

### `display_player_status()`

Muestra un panel con:
- Nombre, nivel y zona actual del jugador.
- Barra de HP con porcentaje.
- Barra de Mana con porcentaje.

---

### `create_status_bar(current, maximum, color, return_values=True)`

Crea una barra de estado visual con caracteres `#` y `-`.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `current` | `float` | Valor actual. |
| `maximum` | `float` | Valor máximo. |
| `color` | `str` | Color para la parte llena (ej: `"red"`, `"blue"`, `"green"`). |
| `return_values` | `bool` | Si `True`, retorna `(bar_str, percentage)`. Si `False`, imprime y retorna `None`. |

**Ejemplo de salida:**

```
[#########################-] 96%
```

---

### `display_main_menu()`

Muestra el menú principal agrupado en 4 secciones: Explorar, Tienda, Personaje, Sistema.

---

### `render_combat_menu(player, enemy, combat_round)`

Muestra la UI de combate con:
- Número de ronda.
- Paneles de HP/MP del jugador y enemigo.
- Tabla de acciones disponibles (Atacar, Usar objeto, Huir).

---

### `display_zone_enemies(zone_name, enemies_data, strings_data)`

Muestra una tabla con los enemigos de una zona, incluyendo HP, daño físico, daño mágico y defensa.

---

### `get_validated_input(prompt, valid_options) -> int`

Solicita input al usuario hasta recibir un número válido dentro de `valid_options`. Ignora Enters residuales.

---

## Funciones de Combate

### `apply_debuffs(player) -> Dict`

Aplica todos los debuffs activos en `player.applieddebuffs`. Para cada debuff, llama a `debuff.Apply(attr, player)` por cada atributo no-callable. Retorna un diccionario de efectos aplicados.

---

## Funciones de Guardado/Carga

### `save_game() -> bool`

Guarda el estado del juego en `Data/SaveGame.json`.

**Datos guardados:**

```json
{
    "player": {
        "name": "...", "level": 1, "health": 100, "health_max": 100,
        "mana": 100, "mana_max": 100, "exp": 15, "exp_max": 100, "gold": 50
    },
    "game_state": { "area": "Forest", "tick": 0 },
    "timestamp": "..."
}
```

> **Nota:** El inventario y el equipamiento **no se guardan** en la implementación actual.

---

### `load_game() -> bool`

Carga el archivo `Data/SaveGame.json` y restaura los atributos básicos del jugador y el estado del juego. Retorna `False` si el archivo no existe o está corrupto.

---

## Bucle Principal (`main`)

La función `main()` se llama en un loop infinito desde el bloque `if __name__ == "__main__"`. En cada iteración:

1. Llama a `player.Check(vInventory)` y regenera el maná.
2. Limpia la pantalla.
3. Muestra el estado del jugador y el menú principal.
4. Solicita y valida el input del usuario.
5. Ejecuta la acción correspondiente.

---

## Opciones del Menú Principal

| Opción | Acción |
|---|---|
| `0` | Salir del juego (con confirmación). |
| `1` | Ver información de zonas (enemigos de Bosque y Cueva en tablas). |
| `2` | Ir a una zona (Bosque, Cueva o salir de la zona actual). Opcionalmente visitar la tienda de zona. |
| `3` | Tienda general con selección entre GeneralStore, WeaponShop, ArmorShop, MagicShop. |
| `4` | Ver inventario (`vInventory.PrintObjects()`). |
| `5` | Usar objetos consumibles del inventario. |
| `6` | Menú de equipamiento (`equip_menu(player, vInventory)`). |
| `7` | Combate contra el enemigo de la zona actual. |
| `8` | Ver estadísticas completas del jugador con barras de HP/MP/EXP y equipamiento actual. |
| `9` | Gestión de mods (listar, recargar, ver estado). |
| `10` | Ver todos los items disponibles en el juego (`item_manager.list_all_items(detailed=True)`). |
| `11` | Guardar la partida. |
| `12` | Menú DEBUG (añadir armas, oro/EXP, ver variables). |

---

### Flujo de Combate (opción 7)

```
¿Hay enemigo en zona?
    NO → Mensaje de error
    SÍ → Bucle de combate:
        ┌─────────────────────────────────────────┐
        │  Mostrar UI combate (render_combat_menu) │
        │  Aplicar debuffs activos                │
        │  Input del jugador (1/2/3)              │
        │                                         │
        │  [1] Atacar:                            │
        │      player.attack(enemy)               │
        │      Si enemy.HP ≤ 0:                   │
        │        → EXP, Gold, loot drop (30%)     │
        │        → Level up si corresponde        │
        │        → Cargar nuevo enemigo            │
        │        → Salir del bucle                │
        │      enemy.attack(player)               │
        │      Si player.HP ≤ 0: Game Over parcial│
        │                                         │
        │  [2] Usar objeto:                       │
        │      Mostrar consumibles                 │
        │      Usar seleccionado                  │
        │      enemy.attack(player) (turno enemigo)│
        │                                         │
        │  [3] Huir:                              │
        │      chance = 0.3 + (AGI_j - AGI_e)/100│
        │      Si éxito: salir del bucle          │
        │      Si falla: enemy ataca con x1.2 dmg │
        └─────────────────────────────────────────┘
```

---

## Ejemplos de Uso

### Ejecutar el juego

```bash
python MainGame.py
```

### Modificar el delay de combate

```python
# En MainGame.py, antes de iniciar:
CONFIG["COMBAT"]["DELAY_BETWEEN_ROUNDS"] = 1.0  # Más rápido
```

### Acceder al item_manager desde el juego

```python
# Ya inicializado en init_game()
item_manager.list_all_items(detailed=True)
```

---

## Notas y Referencias

- **`rich` library:** Usada extensivamente para UI. [https://rich.readthedocs.io/](https://rich.readthedocs.io/)
- **`keyboard` library:** Para capturar Enter en confirmaciones. [https://github.com/boppreh/keyboard](https://github.com/boppreh/keyboard)
- **`Enum` en Python:** Para estados del juego. [https://docs.python.org/3/library/enum.html](https://docs.python.org/3/library/enum.html)
- El sistema de loot usa una tabla hardcodeada por niveles (1-3). Una mejora futura podría cargar la tabla desde el JSON.
- El **guardado no persiste inventario ni equipamiento**. Es una funcionalidad pendiente de implementar.
- La probabilidad de huir está limitada entre 5% y 95%: `flee_chance = max(0.05, min(0.95, flee_chance))`.
