# `ModLoader.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `ModAPI`](#clase-modapi)
3. [Clase `ModInfo`](#clase-modinfo)
4. [Clase `ModLoader`](#clase-modloader)
5. [Variables Globales y Funciones de Acceso](#variables-globales-y-funciones-de-acceso)
6. [Estructura de un Mod](#estructura-de-un-mod)
7. [Ejemplos de Uso](#ejemplos-de-uso)
8. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`ModLoader.py` implementa el **sistema de mods** del juego, similar al sistema de Forge en Minecraft. Permite a desarrolladores externos añadir contenido al juego (items, enemigos, zonas, comandos, eventos) sin modificar el código base.

Los mods se colocan como carpetas dentro del directorio `mods/`, cada una con un archivo `mod.json` de metadatos y un `main.py` con el código del mod.

---

## Clase `ModAPI`

Interfaz que los mods reciben para registrar su contenido en el juego. Es el "contrato" entre el juego y los mods.

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `registered_items` | `dict` | Items registrados por mods. |
| `registered_enemies` | `dict` | Enemigos registrados por mods. |
| `registered_zones` | `dict` | Zonas registradas por mods. |
| `event_handlers` | `dict` | Handlers de eventos organizados por nombre de evento. |
| `custom_commands` | `dict` | Comandos personalizados registrados por mods. |
| `registered_shops` | `dict` | Tiendas registradas por mods. |

### Métodos

#### `register_item(item_or_class, item_data)`

Registra un nuevo item. Acepta tanto una instancia de item como una clase. El ID se toma de `item_data.get('id')`.

```python
api.register_item(mi_espada, {"id": "magic_sword", "name": "Espada Mágica"})
```

---

#### `register_enemy(enemy_or_class, enemy_data)`

Registra un nuevo enemigo. Similar a `register_item`.

```python
api.register_enemy(cEnemy, {"id": "fire_dragon", "NAME": "Dragón de Fuego"})
```

---

#### `register_zone(zone_name, zone_data)`

Registra una nueva zona en el juego.

```python
api.register_zone("dragon_lair", {"name": "Guarida del Dragón", "enemies": [...]})
```

---

#### `register_event_handler(event_name, handler_func)`

Registra una función que se ejecutará cuando ocurra el evento indicado. Múltiples mods pueden registrar handlers para el mismo evento.

**Eventos disponibles (actuales):**

| Nombre del evento | Cuándo se dispara |
|---|---|
| `player_level_up` | Cuando el jugador sube de nivel. |
| `enemy_defeated` | Cuando un enemigo es derrotado. |

```python
def on_level_up(player):
    player.Gold += 50

api.register_event_handler("player_level_up", on_level_up)
```

---

#### `register_command(command_name, handler_func, description="")`

Registra un comando personalizado que puede ser ejecutado desde el juego.

```python
api.register_command("curar", lambda p: setattr(p, "Health", p.Health_max), "Cura completamente")
```

---

#### `register_shop(shop_name, shop_data)`

Registra una nueva tienda.

---

#### `fire_event(event_name, *args, **kwargs)`

Dispara un evento, llamando a todos los handlers registrados para ese nombre. Si un handler lanza una excepción, se registra en el log pero no interrumpe los demás handlers.

---

## Clase `ModInfo`

Objeto de datos que almacena los metadatos de un mod descubierto.

### Atributos

| Atributo | Tipo | Descripción |
|---|---|---|
| `mod_id` | `str` | Identificador único del mod. |
| `name` | `str` | Nombre legible del mod. |
| `version` | `str` | Versión (ej: `"1.0.0"`). |
| `author` | `str` | Nombre del autor. |
| `description` | `str` | Descripción corta. |
| `file_path` | `Path` | Ruta a la carpeta del mod. |
| `loaded` | `bool` | `True` si el mod fue cargado correctamente. |
| `enabled` | `bool` | `True` si el mod está habilitado (siempre `True` por defecto). |

---

## Clase `ModLoader`

Motor principal que descubre, carga y gestiona todos los mods.

### Constructor

```python
ModLoader(mods_directory="mods")
```

Por defecto usa `mods/` relativo a la raíz del proyecto. Si el directorio no existe, lo crea. También crea la estructura del mod de ejemplo (`example_mod/`).

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `mods_directory` | `Path` | Ruta absoluta al directorio de mods. |
| `loaded_mods` | `dict` | Mods cargados: `{mod_id: {"info": ModInfo, "module": Module}}`. |
| `mod_api` | `ModAPI` | Instancia de la API compartida con todos los mods. |
| `mod_configs` | `dict` | Configuraciones de mods (reservado). |

### Métodos

#### `discover_mods() -> dict`

Escanea el directorio de mods. Para cada subcarpeta que contenga `mod.json`, crea un objeto `ModInfo`. Retorna un diccionario `{mod_id: ModInfo}`.

---

#### `load_mod(mod_info: ModInfo) -> bool`

Carga un mod específico:

1. Verifica que exista `main.py`.
2. Usa `importlib.util` para cargar el módulo dinámicamente.
3. Registra el módulo en `sys.modules` como `mod_{mod_id}`.
4. Si el módulo tiene `init_mod(api)`, la llama pasando el `mod_api`.

Retorna `True` si se cargó sin errores.

---

#### `load_all_mods()`

Descubre todos los mods y carga los que estén habilitados. Muestra un resumen con cuántos se cargaron.

---

#### `unload_mod(mod_id) -> bool`

Descarga un mod: llama a `cleanup_mod()` si existe, lo elimina de `sys.modules` y de `loaded_mods`.

---

#### `get_loaded_mods_info() -> List[ModInfo]`

Retorna la lista de `ModInfo` de todos los mods actualmente cargados.

---

#### `display_mods_info()`

Muestra una tabla (con `rich`) de todos los mods cargados con nombre, versión, autor y descripción.

---

#### `_create_example_mod_structure()`

*(Interno)* Crea la carpeta `mods/example_mod/` con `mod.json`, `main.py` y `README.md` de plantilla al iniciar si no existe. Si detecta un `main.py` con el sistema de daño antiguo, lo actualiza automáticamente.

---

## Variables Globales y Funciones de Acceso

| Variable | Tipo | Descripción |
|---|---|---|
| `mod_loader` | `Optional[ModLoader]` | Instancia global Singleton del `ModLoader`. |

### `get_mod_loader() -> ModLoader`

Patrón Singleton. Retorna la instancia global del `ModLoader`.

### `get_mod_api() -> ModAPI`

Atajo para obtener la `ModAPI` de la instancia global.

---

## Estructura de un Mod

Para crear un mod, crea una carpeta en `mods/` con esta estructura:

```
mods/
└── mi_mod/
    ├── mod.json        ← Metadatos obligatorios
    ├── main.py         ← Código del mod
    └── README.md       ← Documentación (opcional)
```

### `mod.json`

```json
{
    "mod_id": "mi_mod",
    "name": "Mi Mod",
    "version": "1.0.0",
    "author": "Tu Nombre",
    "description": "Descripción de mi mod",
    "main": "main.py",
    "dependencies": [],
    "game_version": "1.0.0"
}
```

### `main.py` (plantilla mínima)

```python
from Modules.setup import cWeapon, cEnemy

def init_mod(api):
    """Llamada automáticamente cuando el mod se carga."""
    register_items(api)

def register_items(api):
    nueva_espada = cWeapon(
        ID=1001,       # IDs de mods empiezan en 1000+
        name="Espada del Mod",
        DMG=[40, 10, 0, 0, 0],
        DMG_min=[25, 5, 0, 0],
        Gold_Cost=500,
        Description="Una espada añadida por un mod"
    )
    api.register_item(nueva_espada, {"id": "mod_sword"})

def cleanup_mod():
    """Opcional: se llama al descargar el mod."""
    print("Mod limpiado")
```

---

## Ejemplos de Uso

### Cargar todos los mods al iniciar el juego

```python
from Modules.ModLoader import get_mod_loader

mod_loader = get_mod_loader()
mod_loader.load_all_mods()
```

### Disparar un evento desde el juego

```python
from Modules.ModLoader import get_mod_api

api = get_mod_api()
api.fire_event("player_level_up", player)
api.fire_event("enemy_defeated", player, enemy)
```

### Ver mods cargados

```python
from Modules.ModLoader import get_mod_loader

loader = get_mod_loader()
loader.display_mods_info()
```

---

## Notas y Referencias

- **`importlib.util`:** Módulo para importación dinámica de módulos Python. Documentación: [https://docs.python.org/3/library/importlib.html#importlib.util.spec_from_file_location](https://docs.python.org/3/library/importlib.html#importlib.util.spec_from_file_location)
- **`sys.modules`:** Diccionario que Python usa para cachear módulos importados. Al registrar un módulo aquí, se hace disponible globalmente. [https://docs.python.org/3/reference/import.html#the-module-cache](https://docs.python.org/3/reference/import.html#the-module-cache)
- Los IDs de items de mods deben empezar en 1000+ para no colisionar con los items base del juego.
- Si `rich` no está instalado, el `ModLoader` usa una clase de consola de respaldo que imprime sin formato, garantizando que el sistema funcione incluso sin la dependencia.
