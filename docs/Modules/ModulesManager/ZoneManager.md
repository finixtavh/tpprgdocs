# `ZoneManager.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)

---

## 1. Descripción General
`ZoneManager.py` es el gestor dinámico de Zonas de TPPRPG. Se encarga de parsear y normalizar el contenido del archivo `DataStats.json` y coordinar con el `ModLoader` para inyectar áreas nuevas creadas por mods. Su función principal es proveer al motor principal la lista de zonas accesibles, sus descripciones, colores de interfaz y los enemigos que habitan en ellas.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere el JSON `./Data/DataStats.json` para definir las zonas base (como `Forest`, `Castle`, etc.).
- **Inyecciones**: 
  - Importa dinámicamente `get_mod_api()` desde `ModulesUtils.ModLoader` para recoger zonas inyectadas por los mods.
  - Utiliza la función `Read` de `ModulesUtils.Loader` para la lectura segura de archivos JSON.

---

## 3. Constantes y Variables Globales
- `_zone_manager_instance` (`ZoneManager` | `None`): Instancia Singleton global para centralizar la carga de zonas.

---

## 4. Clases y Estructuras de Datos

### `ZoneManager`
Gestor principal del sistema de zonas.

#### Atributos de Instancia
- `self.datastats_path` (`str`): Ruta del archivo (`"./Data/DataStats.json"`).
- `self.zones_cache` (`Dict`): Diccionario para caché interna (usado parcialmente en implementaciones futuras para evitar llamadas redundantes a modding).

#### Métodos Principales

- `__init__(self, datastats_path: str = "./Data/DataStats.json")`
  - **Propósito**: Inicializa el gestor.

- `get_all_zones(self) -> Dict[str, Dict]`
  - **Propósito**: Consolida las zonas del juego base y las zonas de los mods en un solo diccionario gigante.
  - **Retornos**: Un diccionario donde cada llave es el ID de la zona y su valor es un diccionario normalizado con `name`, `desc`, `color`, `enemies`, `gathering`, y `source`.

- `_load_zones_from_json(self) -> Dict[str, Dict]`
  - **Propósito**: Llama a la función `Read` para parsear todo el `DataStats.json`. Itera sobre las llaves principales validando que contengan un array. Llama a `_extract_zone_info` para cada una y le asigna el `source` de `"base"`.

- `_load_zones_from_mods(self) -> Dict[str, Dict]`
  - **Propósito**: Pide al `ModAPI` la lista de zonas `stats_injections`. Las valida y las parsea con `_extract_zone_info`, asignándoles el `source` de `"mod"`. Posee un `try/except` silencioso para no crashear si el `ModLoader` no está iniciado.

- `_extract_zone_info(self, zone_key: str, zone_data: Dict, source: str) -> Dict`
  - **Propósito**: Extrae y normaliza los strings (Nombre, Descripción, Mensaje de Salida, Color de Consola) y aísla los diccionarios de Enemigos y la metadata de `Gathering` (Recolección).
  - **Retornos**: Un diccionario estandarizado de la zona. Retorna `None` si la zona está malformada.

- `get_zone_by_key(self, zone_key: str) -> Dict`
  - **Propósito**: Busca una zona en específico (ej: `"Forest"`) en el conjunto total de zonas devueltas por `get_all_zones()`.

- `get_zone_enemies(self, zone_key: str) -> List`
  - **Propósito**: Devuelve exclusivamente el array de enemigos de una zona específica.

- `get_zones_list(self) -> List[Tuple[str, str, str]]`
  - **Propósito**: Prepara una lista optimizada para menús interactivos de selección.
  - **Retornos**: Una lista de tuplas `(zone_key, zone_name, zone_color)`.

---

## 5. Funciones del Módulo (API)

El módulo exporta funciones *helper* globales para que otros sistemas no tengan que instanciar o buscar el Singleton manualmente:

- `get_zone_manager(datastats_path: str = "./Data/DataStats.json") -> ZoneManager`
  - **Propósito**: Patrón Singleton. Retorna la instancia global, creándola si es necesario.

- `get_all_zones() -> Dict[str, Dict]`
  - **Propósito**: Wrapper global para `manager.get_all_zones()`.

- `get_zones_for_selector() -> List[Tuple[str, str, str]]`
  - **Propósito**: Wrapper global para `manager.get_zones_list()`.

- `get_zone_info(zone_key: str) -> Dict`
  - **Propósito**: Wrapper global para `manager.get_zone_by_key(zone_key)`.
