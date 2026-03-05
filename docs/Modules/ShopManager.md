# `ShopManager.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `ShopManager`](#clase-shopmanager)
3. [Variables Globales](#variables-globales)
4. [Funciones Helper](#funciones-helper)
5. [Ejemplos de Uso](#ejemplos-de-uso)
6. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`ShopManager.py` gestiona las **tiendas del juego** cargando su inventario desde el archivo `DataShops.json`. Permite consultar, agregar y remover items de tiendas de forma persistente, además de mostrar una interfaz interactiva de compra en la consola.

---

## Clase `ShopManager`

Gestor de tiendas. Lee y escribe la configuración de tiendas desde/hacia JSON.

### Constructor

```python
ShopManager(shops_json_path="./Data/DataShops.json")
```

### Atributos de instancia

| Atributo | Tipo | Descripción |
|---|---|---|
| `shops_json_path` | `str` | Ruta al archivo JSON de configuración de tiendas. |
| `shops_data` | `dict` | Datos de todas las tiendas en memoria. |

---

### Métodos

#### `load_shops() -> bool`

Carga las tiendas desde el JSON. Si el archivo no existe, llama a `_create_default_shops_file()` para generar uno con tiendas de ejemplo. Retorna `True` si se cargó correctamente.

---

#### `get_shop_items(shop_name: str) -> List[int]`

Retorna la lista de IDs de items disponibles en la tienda especificada.

```python
ids = manager.get_shop_items("WeaponShop")
# [1, 2, 3, 4]
```

---

#### `add_item_to_shop(shop_name: str, item_id: int) -> bool`

Agrega un item (por ID) al inventario de la tienda. Guarda automáticamente los cambios en JSON. No agrega duplicados.

---

#### `remove_item_from_shop(shop_name: str, item_id: int) -> bool`

Remueve un item del inventario de la tienda y guarda los cambios.

---

#### `save_shops() -> bool`

Persiste el estado actual de `shops_data` en el archivo JSON con formato indentado.

---

#### `_create_default_shops_file()`

*(Método interno)* Crea el archivo JSON con las tiendas por defecto:

| Tienda | IDs de items |
|---|---|
| `GeneralStore` | 1, 10, 20, 30, 40, 50, 51, 53 |
| `WeaponShop` | 1, 2, 3, 4 |
| `ArmorShop` | 10, 11, 12, 20, 21, 30, 31 |
| `MagicShop` | 4, 5, 41, 42, 43, 44, 54, 55 |

---

## Variables Globales

| Variable | Tipo | Descripción |
|---|---|---|
| `_shop_manager_instance` | `Optional[ShopManager]` | Instancia global Singleton del `ShopManager`. |

---

## Funciones Helper

### `get_shop_manager(shops_path) -> ShopManager`

Patrón **Singleton**. Retorna la instancia global del `ShopManager`. La primera vez la crea y llama a `load_shops()` automáticamente.

```python
manager = get_shop_manager()
```

---

### `display_shop(shop_name, player, item_manager)`

Muestra la interfaz de tienda interactiva en la consola. Permite al jugador ver items disponibles, sus stats y precios, y comprarlos.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `shop_name` | `str` | Nombre de la tienda a mostrar (ej: `"GeneralStore"`). |
| `player` | `cPlayer` | El objeto del jugador (para verificar/descontar oro). |
| `item_manager` | `ItemManager` | Instancia del gestor de items para cargar datos del item. |

**Flujo de la función:**

1. Obtiene los IDs de items de la tienda.
2. Muestra una tabla con nombre, costo, tipo y disponibilidad (✓/✗ según el oro del jugador).
3. El jugador selecciona un número para ver detalles del item.
4. Confirma la compra (`s/n`).
5. Si confirma y tiene oro suficiente: descuenta el oro, crea el objeto del item y lo añade al inventario.

**Stats mostrados:**

- Daño Físico (de `DMG`)
- Daño Mágico (de `Magical_DMG`)
- Defensa (`Defense`)
- HP Máximo (`HealthBoost`)
- Daño bonus (`DmgBoost`)
- Robo de Vida (`HealthSteal`)
- Restauración de HP/MP (`HealthRestore`, `ManaRestore`)

---

## Ejemplos de Uso

### Inicializar y consultar tienda

```python
from Modules.ShopManager import get_shop_manager

manager = get_shop_manager()
item_ids = manager.get_shop_items("WeaponShop")
print(item_ids)  # [1, 2, 3, 4]
```

### Agregar un item a una tienda

```python
manager = get_shop_manager()
manager.add_item_to_shop("WeaponShop", 99)
# Guarda automáticamente en DataShops.json
```

### Mostrar la tienda interactiva

```python
from Modules.ShopManager import display_shop
from Modules.ItemManager import get_item_manager

item_manager = get_item_manager()
display_shop("GeneralStore", player, item_manager)
```

### Estructura esperada en `DataShops.json`

```json
{
    "GeneralStore": [
        {
            "IDs": [1, 10, 20, 30, 40, 50]
        }
    ],
    "WeaponShop": [
        {
            "IDs": [1, 2, 3, 4]
        }
    ]
}
```

---

## Notas y Referencias

- La función `display_shop` importa `keyboard` para manejar la espera de confirmación. Esta librería requiere instalación: `pip install keyboard`.
- Los IDs en el JSON deben coincidir con los `STATIC_ID` definidos en `DataItems.json`, que maneja el `ItemManager`.
- **`keyboard` library:** Permite detectar pulsaciones de teclas. Documentación: [https://github.com/boppreh/keyboard](https://github.com/boppreh/keyboard)
- En Linux, la librería `keyboard` puede requerir permisos de root para funcionar (de ahí la integración con `check_admin.py`).
