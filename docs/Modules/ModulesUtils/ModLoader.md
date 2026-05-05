# `ModLoader.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Gestor: `ModLoader`](#gestor-modloader)
3. [API de Modding (`ModAPI`)](#api-de-modding-modapi)
4. [Estructura de un Mod](#estructura-de-un-mod)
5. [Inyección de Datos](#inyección-de-datos)
6. [Eventos y Comandos](#eventos-y-comandos)
7. [Ejemplo de Mod](#ejemplo-de-mod)

---

## Descripción General

`ModLoader.py` es el motor de extensibilidad del juego. Permite a terceros añadir contenido (ítems, enemigos, zonas, misiones, mecánicas de gremio) sin modificar el código base, mediante la carga dinámica de scripts Python y archivos de datos JSON.

---

## Gestor: `ModLoader`

Se encarga del ciclo de vida de los mods.
- **`discover_mods()`**: Escanea la carpeta `mods/` en busca de subcarpetas que contengan archivos `.py` y `.json` con el mismo nombre.
- **`load_all_mods()`**: Carga todos los mods encontrados, mostrando una barra de progreso visual.
- **`unload_mod(name)`**: Descarga un mod y libera sus recursos (opcionalmente llamando a `cleanup_mod`).

---

## API de Modding (`ModAPI`)

Es el objeto que se pasa a los mods para que puedan interactuar con el juego. Proporciona métodos para registrar contenido nuevo.

### Funciones de Inyección
- `auto_inject_from_mod_data()`: Lee automáticamente el JSON del mod e inyecta ítems, zonas y tiendas en los managers correspondientes.
- `register_guild_skill()`: Añade nuevas habilidades al árbol de gremios.
- `register_weather()`: Introduce nuevos tipos de clima con sus respectivos modificadores.
- `register_player_attribute()`: Permite guardar variables personalizadas en el objeto `player` que persistirán en el guardado.

---

## Estructura de un Mod

Un mod debe estar en su propia carpeta dentro de `mods/`:
```text
mods/
  mi_super_mod/
    mi_super_mod.py   # Lógica
    mi_super_mod.json # Datos
```

---

## Inyección de Datos

El sistema de inyección permite que el mod defina contenido en su JSON y el juego lo asuma como propio:
- **Items**: Inyectados en `ItemManager`.
- **Zonas**: Inyectadas en `ZoneManager` (procedentes de `DataStats.json`).
- **Tiendas**: Inyectadas en `ShopManager`.
- **Recetas**: Inyectadas en `CraftingSystem`.

---

## Eventos y Comandos

- **Eventos**: Los mods pueden suscribirse a eventos como `on_combat_start` o `on_item_crafted` mediante `register_event_handler`.
- **Comandos**: Permite registrar comandos de consola personalizados que el jugador puede ejecutar.
- **Tick Handlers**: Funciones que se ejecutan en cada paso de la simulación del mundo (especialmente útil para mods de gremios).

---

## Ejemplo de Mod

### `mi_mod.py`
```python
def init_mod(api, data):
    print("¡Mod de prueba cargado!")
    
    # Registrar un comando nuevo
    api.register_command("saludar", lambda: print("¡Hola desde el mod!"))

def cleanup_mod():
    print("Limpiando recursos del mod...")
```

---

## Notas Técnicas

- Utiliza `importlib.util` para cargar módulos Python en tiempo de ejecución.
- Posee un sistema de **Fallback Console** para seguir funcionando incluso si la librería `rich` no está instalada (aunque es un requisito del motor).
- Mantiene un log independiente en `tpprpg_mods.log` para facilitar la depuración de errores en mods.
