# `ZoneManager.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `ZoneManager`](#clase-zonemanager)
3. [Variables Globales y Funciones de Acceso](#variables-globales-y-funciones-de-acceso)
4. [Funciones Helper](#funciones-helper)
5. [Estructura de una Zona](#estructura-de-una-zona)
6. [Ejemplos de Uso](#ejemplos-de-uso)
7. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`ZoneManager.py` gestiona de forma dinámica todas las **zonas del mundo del juego**. Carga las zonas base desde `DataStats.json` y las combina con zonas añadidas por mods (a través del `ModAPI`), ofreciendo una API unificada para acceder a enemigos, recolectables y metadatos de cada zona.

El sistema sigue el patrón **Singleton** para mantener una única instancia durante toda la partida.

---

## Clase `ZoneManager`

### Constructor

```python
ZoneManager(datastats_path="./Data/DataStats.json")
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `datastats_path` | `str` | Ruta al archivo JSON base de zonas y estadísticas. |
| `zones_cache` | `dict` | Caché de zonas (reservado para futuras optimizaciones; actualmente no se usa). |

---

### Métodos

#### `get_all_zones() -> Dict[str, Dict]`

Punto de entrada principal. Combina zonas base y zonas de mods en un único diccionario.

**Flujo interno:**
1. Llama a `_load_zones_from_json()` para obtener zonas del JSON base.
2. Llama a `_load_zones_from_mods()` para obtener zonas añadidas por mods.
3. Las zonas de mods sobreescriben las base si tienen el mismo `zone_key`.

**Retorno:** Diccionario con formato:
```python
{
    "ZoneKey": {
        "name":      "Nombre mostrado",
        "desc":      "Descripción de la zona",
        "exit":      "Texto al salir de la zona",
        "color":     "green",
        "enemies":   [...],       # Lista de dicts de enemigos
        "gathering": {...},       # Datos de recolección
        "source":    "base"       # "base" o "mod"
    }
}
```

---

#### `get_zone_by_key(zone_key: str) -> Dict`

Retorna la información completa de una zona específica. Retorna `None` si la zona no existe.

```python
zona = manager.get_zone_by_key("Forest")
```

---

#### `get_zone_enemies(zone_key: str) -> List`

Retorna la lista de dicts de enemigos de una zona. Retorna lista vacía si la zona no existe.

```python
enemigos = manager.get_zone_enemies("Cave")
```

---

#### `get_zones_list() -> List[Tuple[str, str, str]]`

Retorna una lista de tuplas con la información mínima de cada zona, útil para construir el selector de zonas en la UI.

**Retorno:** Lista de tuplas `(zone_key, zone_name, zone_color)`.

```python
zonas = manager.get_zones_list()
# [("Forest", "Bosque", "green"), ("Cave", "Cueva", "grey"), ...]
```

---

#### `_load_zones_from_json() -> Dict[str, Dict]`

*(Método interno)* Lee el archivo `DataStats.json` completo usando `Read()` del módulo `Loader`. Itera sobre todas las claves del JSON y, para cada entrada que sea un array válido, extrae la información de zona con `_extract_zone_info()`.

---

#### `_load_zones_from_mods() -> Dict[str, Dict]`

*(Método interno)* Obtiene zonas registradas por mods desde `ModAPI.get_all_zones()`. Si el `ModLoader` no está disponible, retorna silenciosamente un dict vacío (sin lanzar error).

---

#### `_extract_zone_info(zone_key, zone_data, source) -> Dict`

*(Método interno)* Extrae y estructura la información de una zona desde los datos raw del JSON.

**Lógica de extracción:**

- Busca el objeto `"Strings"` para obtener `name`, `desc`, `exit` y `color`.
- Considera enemigos a todas las claves que no sean `"Strings"` ni `"Gathering"` y que sean un `dict` con la clave `"NAME"`.
- Extrae `"Gathering"` directamente si existe.
- Retorna `None` si no hay objeto `"Strings"` (la entrada no es una zona válida).

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `zone_key` | `str` | Clave de la zona en el JSON (ej: `"Forest"`, `"Cave"`). |
| `zone_data` | `dict` | Datos de la zona (primer elemento del array en el JSON). |
| `source` | `str` | Origen: `"base"` para JSON base, `"mod"` para mods. |

---

## Variables Globales y Funciones de Acceso

| Variable | Tipo | Descripción |
|---|---|---|
| `_zone_manager_instance` | `ZoneManager \| None` | Instancia global Singleton del `ZoneManager`. |

### `get_zone_manager(datastats_path) -> ZoneManager`

Patrón **Singleton**. Crea la instancia la primera vez y la retorna en llamadas sucesivas.

```python
from Modules.ZoneManager import get_zone_manager
manager = get_zone_manager()
```

---

## Funciones Helper

### `get_all_zones() -> Dict[str, Dict]`

Atajo para obtener todas las zonas disponibles.

```python
from Modules.ZoneManager import get_all_zones
zonas = get_all_zones()
```

---

### `get_zones_for_selector() -> List[Tuple[str, str, str]]`

Atajo para obtener la lista de zonas en formato `(key, name, color)` para el selector de UI.

```python
from Modules.ZoneManager import get_zones_for_selector
lista = get_zones_for_selector()
```

---

### `get_zone_info(zone_key: str) -> Dict`

Atajo para obtener los datos completos de una zona específica.

```python
from Modules.ZoneManager import get_zone_info
info = get_zone_info("Forest")
print(info["name"])    # "Bosque"
print(info["color"])   # "green"
print(info["enemies"]) # [{"NAME": "Goblin", ...}, ...]
```

---

## Estructura de una Zona

La estructura esperada en `DataStats.json` para una zona es:

```json
{
    "Forest": [
        {
            "Strings": {
                "name":  "Bosque",
                "desc":  "Un denso bosque lleno de criaturas.",
                "exit":  "Saliendo del Bosque...",
                "color": "green"
            },
            "Goblin": {
                "NAME":    "Goblin",
                "HP_M":    40,
                "DMG":     [8, 0, 0, 0, 0],
                "DMG_min": [4, 0, 0, 0],
                "DEF":     [2, 0, 0, 0, 0],
                "EXP":     15,
                "GOLD":    10
            },
            "Wolf": {
                "NAME":    "Lobo",
                "HP_M":    55,
                "DMG":     [12, 0, 0, 0, 0],
                "DMG_min": [7, 0, 0, 0],
                "DEF":     [1, 0, 0, 0, 0],
                "EXP":     20,
                "GOLD":    8
            },
            "Gathering": {
                "Materials": ["1_G", "2_G"]
            }
        }
    ]
}
```

**Campos del objeto `Strings`:**

| Campo | Tipo | Descripción |
|---|---|---|
| `name` | `str` | Nombre mostrado al jugador. |
| `desc` | `str` | Descripción de la zona. |
| `exit` | `str` | Mensaje mostrado al salir. Default: `"Saliendo de {zone_key}"`. |
| `color` | `str` | Color Rich para el nombre de la zona en menús. |

---

## Ejemplos de Uso

### Obtener enemigos de una zona para el combate

```python
from Modules.ZoneManager import get_zone_manager
import random

manager  = get_zone_manager()
enemigos = manager.get_zone_enemies("Forest")

if enemigos:
    enemigo_data = random.choice(enemigos)
    print(f"Enemigo encontrado: {enemigo_data['NAME']}")
```

### Construir el selector de zona dinámicamente

```python
from Modules.ZoneManager import get_zones_for_selector
from Modules.input_utils import interactive_menu_select

zonas = get_zones_for_selector()

opciones = [(i + 1, f"[{info[2]}]{info[1]}[/{info[2]}]") for i, info in enumerate(zonas)]
opciones.append((0, "Volver"))

eleccion = interactive_menu_select(
    title="Seleccionar Zona",
    options=opciones,
    border_style="green",
)
```

### Verificar si una zona tiene material de recolección

```python
from Modules.ZoneManager import get_zone_info

info = get_zone_info("Forest")
if info and info.get("gathering"):
    materiales = info["gathering"].get("Materials", [])
    print(f"Materiales disponibles: {materiales}")
```

---

## Notas y Referencias

- El método `_load_zones_from_mods()` captura silenciosamente cualquier excepción si el `ModLoader` no está inicializado, lo que hace que el `ZoneManager` funcione aunque no haya mods cargados.
- Las zonas de mods **sobreescriben** las del JSON base si usan la misma `zone_key`. Esto permite a los mods modificar zonas existentes.
- **`Loader.Read`:** Se usa para leer el JSON completo pasando `None` como segundo argumento. Ver `Loader.md` para más detalles.
- **`ModAPI.get_all_zones()`:** Método que retorna el diccionario `stats_injections` de la `ModAPI`, donde los mods registran sus zonas. Ver `ModLoader.md`.
