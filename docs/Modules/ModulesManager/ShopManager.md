# `ShopManager.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Gestor: `ShopManager`](#gestor-shopmanager)
3. [Estructura del JSON (`DataShops.json`)](#estructura-del-json-datashopsjson)
4. [Interfaz: Menú de Tienda](#interfaz-menú-de-tienda)
5. [Proceso de Compra](#proceso-de-compra)
6. [Gestión de Inventario de Tiendas](#gestión-de-inventario-de-tiendas)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`ShopManager.py` gestiona los establecimientos comerciales del juego. Permite definir múltiples tiendas (Armerías, Boticas, Tiendas Generales) con inventarios independientes cargados desde archivos JSON, y maneja la lógica de transacción con el jugador.

---

## Gestor: `ShopManager`

Carga la configuración de las tiendas y proporciona métodos para consultar qué ítems vende cada una.

### Funciones Principales
- `load_shops()`: Inicializa el catálogo de tiendas.
- `get_shop_items(shop_name)`: Retorna la lista de IDs de ítems que vende una tienda específica.
- `save_shops()`: Persiste cambios en la configuración de las tiendas.

---

## Estructura del JSON (`DataShops.json`)

El archivo organiza las tiendas por nombre, cada una conteniendo un array de IDs de ítems (vinculados a los IDs del `ItemManager`).

```json
{
    "GeneralStore": [{ "IDs": [1, 10, 20] }],
    "WeaponShop": [{ "IDs": [1, 2, 3] }]
}
```
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)

---

## 1. Descripción General
El `ShopManager.py` es responsable de la carga, gestión y visualización de las tiendas dentro del juego. Carga la configuración de inventario de las distintas tiendas desde un archivo JSON y proporciona una interfaz de consola interactiva para que el jugador (mediante `display_shop`) pueda previsualizar estadísticas y comprar objetos.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere `./Data/DataShops.json` para el inventario.
- **Inyecciones**: 
  - Utiliza `interactive_menu_select` y `wait_for_enter` de `Modules.ModulesUtils.input_utils` para la UI.
  - Al instanciar objetos comprados, inyecta `cWeapon`, `cEquippableItems`, `cObject` (de `setup.py`) y `cTool` (de `GatheringMasterySystem`).
  - Utiliza globalmente el `ItemManager` para resolver los IDs a objetos reales.

---

## 3. Constantes y Variables Globales
- `_shop_manager_instance` (`ShopManager` | `None`): Instancia Singleton global para evitar recargas constantes del JSON.

---

## 4. Clases y Estructuras de Datos

### `ShopManager`
Gestor principal del sistema de tiendas.

#### Atributos de Instancia
- `self.shops_json_path` (`str`): Ruta del archivo de configuración de tiendas (por defecto `"./Data/DataShops.json"`).
- `self.shops_data` (`Dict`): Diccionario interno que mantiene el stock (lista de IDs) de todas las tiendas. Su estructura típica es:
  ```json
  { "GeneralStore": [ { "IDs": [1, 10, 20] } ] }
  ```

#### Métodos Principales

- `__init__(self, shops_json_path: str = "./Data/DataShops.json")`
  - **Propósito**: Inicializa el gestor.

- `load_shops(self) -> bool`
  - **Propósito**: Carga la configuración de tiendas desde disco. Si el archivo no existe, llama a `_create_default_shops_file()`.
  - **Retornos**: `True` si la carga es exitosa.

- `get_shop_items(self, shop_name: str) -> List[int]`
  - **Propósito**: Devuelve la lista de `STATIC_ID` que vende una tienda específica.
  - **Parámetros**: `shop_name` (ej. `"WeaponShop"`).
  - **Retornos**: Lista de IDs (`List[int]`). Si la tienda no existe o está vacía, devuelve `[]`.

- `add_item_to_shop(self, shop_name: str, item_id: int) -> bool`
  - **Propósito**: Añade dinámicamente un ID al inventario de una tienda y guarda los cambios permanentemente.
  - **Parámetros**: `shop_name` (Nombre de la tienda), `item_id` (ID del objeto).
  - **Retornos**: `True` si se añadió con éxito. `False` si el ítem ya existía en la tienda.

- `remove_item_from_shop(self, shop_name: str, item_id: int) -> bool`
  - **Propósito**: Quita un objeto del inventario de la tienda permanentemente.

- `save_shops(self) -> bool`
  - **Propósito**: Vuelca el estado de `self.shops_data` al archivo JSON.

- `_create_default_shops_file(self)`
  - **Propósito**: Crea el archivo `DataShops.json` con configuraciones por defecto (`GeneralStore`, `WeaponShop`, `ArmorShop`, `MagicShop`) si el archivo es eliminado o no se encuentra.

---

## 5. Funciones del Módulo (API)

- `get_shop_manager(shops_path: str = "./Data/DataShops.json") -> ShopManager`
  - **Propósito**: Devuelve el Singleton del `ShopManager`. Asegura que las tiendas estén cargadas en memoria.

- `display_shop(shop_name: str, player, item_manager)`
  - **Propósito**: Interfaz principal (UI) para interactuar con una tienda.
  - **Parámetros**: 
    - `shop_name` (`str`): El identificador de la tienda a mostrar.
    - `player` (`cPlayer`): Referencia al jugador actual (para descontar oro y validar fondos).
    - `item_manager` (`ItemManager`): Instancia del gestor de ítems para resolver los nombres, costos y estadísticas de cada ID.
  - **Comportamiento**:
    1. Extrae los ítems de `shop_name`.
    2. Renderiza una lista mostrando los nombres, costos y si el jugador tiene suficiente oro (indicado con `✓` o `✗`).
    3. Cuando el jugador selecciona un objeto, muestra un desglose de todas sus estadísticas (Daño por tipo, Defensas, Robo de vida, Curación, etc.).
    4. Pide confirmación de compra. Si se confirma, resta el oro del `player`, instancia el objeto a través del `item_manager` y lo empuja al inventario global (`vInventory`).
---

## Notas Técnicas

- Los precios de venta se obtienen directamente del campo `Gold_Cost` definido en `DataItems.json`.
