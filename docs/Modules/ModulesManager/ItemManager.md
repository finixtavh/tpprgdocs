# `ItemManager.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)

---

## 1. Descripción General
`ItemManager.py` es el pilar central que gestiona la base de datos de todos los objetos del juego. Actúa como un motor de persistencia y una factoría que transforma datos planos de JSON (`DataItems.json`) en objetos complejos de Python con los que el motor de juego puede interactuar, resolviendo la indexación tanto por `ID` estático como por nombre de objeto.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere el archivo de configuración base `./Data/DataItems.json`.
- **Inyecciones Condicionales**: Las funciones de factoría pueden importar dinámicamente o recibir dependencias de `cWeapon`, `cEquippableItems`, `cObject`, `cTool`, y `cMaterial` provenientes de varios sistemas (ej. `GatheringMasterySystem.py`, `setup.py`, `MainGame.py`).

---

## 3. Constantes y Variables Globales
- `_item_manager_instance` (`ItemManager` | `None`): Instancia única Singleton del gestor de ítems. Se inicializa y cachea globalmente para evitar recargas constantes del disco duro durante el transcurso del juego.

---

## 4. Clases y Estructuras de Datos

### `ItemManager`
Maneja la carga, indexación, creación y guardado (CRUD) de ítems.

#### Atributos de Instancia
- `self.items_json_path` (`str`): La ruta de guardado/lectura del archivo (por defecto `"./Data/DataItems.json"`).
- `self.items_data` (`Dict`): El diccionario crudo cargado directamente del archivo JSON con toda su estructura de categorías.
- `self.items_by_id` (`Dict[int, Dict]`): Diccionario que funciona como índice para acceso rápido mediante ID (`STATIC_ID`). Los materiales utilizan prefijos (`"mat_ID"`) para evitar colisiones.
- `self.items_by_name` (`Dict[str, Dict]`): Índice para acceso O(1) de ítems mediante su nombre en minúsculas (búsqueda case-insensitive).

#### Métodos Principales

- `__init__(self, items_json_path: str = "./Data/DataItems.json")`
  - **Propósito**: Constructor de la clase.

- `load_items(self) -> bool`
  - **Propósito**: Carga todos los items desde el archivo JSON al diccionario interno e inicia la indexación.
  - **Retornos**: `True` si se cargó correctamente y parseó el JSON, `False` si el archivo no existe o tuvo un error. Llama a `_create_default_items_file()` si el archivo no existe.

- `_build_indexes(self)`
  - **Propósito**: Construye los diccionarios paralelos `items_by_id` y `items_by_name`. Los objetos de categoría `TYPE:Materials` son indexados usando su `MATERIAL_ID` con un sufijo de salvaguarda `mat_`.

- `get_item_by_id(self, item_id: Union[int, str], material: bool = False) -> Optional[Dict]`
  - **Propósito**: Busca los datos crudos del objeto.
  - **Parámetros**: `item_id` (Entero del ID), `material` (Booleano. Si es True busca como `mat_{id}`).

- `get_item_by_name(self, name: str) -> Optional[Dict]`
  - **Propósito**: Retorna la data JSON de un item (case-insensitive).

- `get_items_by_type(self, item_type: str) -> List[Dict]`
  - **Propósito**: Retorna una lista con todos los objetos que pertenezcan a la categoría proveída (ej. `"Weapon"`, buscará `"TYPE:Weapon"`).

- `create_item_object(self, item_id: Union[int, str], cWeapon=None, cEquippableItems=None, cObject=None, cTool=None, cMaterial=None, material_lookup: bool = False)`
  - **Propósito**: Factoría principal de instanciado. Pasa un ID o nombre por los índices y devuelve una instancia de Python del objeto.
  - **Parámetros**: Dependencias de las Clases a instanciar (ya que este módulo no las importa todas para evitar referencias circulares). 
  - **Retornos**: Objeto instanciado (`cWeapon`, `cMaterial`, etc.).

#### Métodos de Creación Internos (Factory Wrappers)
- `_create_weapon(self, item_data: Dict, cWeapon)`
- `_create_equippable(self, item_data: Dict, cEquippableItems)`
- `_create_consumable(self, item_data: Dict, cObject)`
- `_create_tool(self, item_data: Dict, cTool)`
- `_create_material(self, item_data: Dict, cMaterial)`
  - **Propósito**: Interpretan el diccionario JSON del ítem y retornan las clases base. Controlan atributos como la matriz de Daño (`[Físico, Térmico, Tierra, Eléctrico, Profundo]`).

#### Operaciones CRUD (Manipulación y Guardado)
- `add_item(self, item_type: str, item_key: str, item_data: Dict) -> bool`
  - **Propósito**: Agrega un nuevo ítem a `items_data`, reconstruye índices y guarda en el JSON de disco.
- `modify_item(self, item_type: str, item_key: str, modifications: Dict) -> bool`
  - **Propósito**: Aplica mutaciones en lote a las propiedades de un ítem existente y guarda los cambios permanentemente.
- `delete_item(self, item_type: str, item_key: str) -> bool`
  - **Propósito**: Elimina por completo un ítem del juego.

#### Métodos de Impresión UI (Rich)
- `list_all_items(self, category: Optional[str] = None, detailed: bool = False, discovered_ids: set = None, limit: int = 50)`
  - **Propósito**: Utiliza `rich.console` para imprimir una tabla estilizada de todos los objetos en el JSON. Admite censura con caracteres de bloque (`████`) si se le pasa un set de `discovered_ids` y el ítem no ha sido descubierto por el jugador aún.

---

## 5. Funciones del Módulo (API)

- `get_item_manager() -> ItemManager`
  - **Propósito**: Provee acceso seguro al Singleton del ItemManager. Instancia y carga la base de datos de disco en la primera llamada de ejecución del juego, devolviendo siempre el objeto cacheado en llamadas posteriores.
