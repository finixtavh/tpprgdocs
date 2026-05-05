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

---

## Interfaz: Menú de Tienda

### `display_shop(shop_name, player, item_manager)`
Muestra una interfaz interactiva donde el jugador puede:
1. Ver la lista de ítems a la venta con sus precios.
2. Identificar visualmente si puede permitirse el objeto (`✓` o `✗`).
3. Ver el oro actual disponible.
4. Consultar detalles técnicos de un ítem (Daño, Defensa, Descripción) antes de comprar.

---

## Proceso de Compra

Cuando el jugador selecciona un ítem para comprar:
1. **Validación**: Se comprueba si el jugador tiene suficiente oro.
2. **Confirmación**: Se pide una última confirmación al jugador.
3. **Creación**: El `ItemManager` instancia el objeto real a partir del ID.
4. **Transacción**: Se descuenta el oro y se añade el objeto al inventario global `vInventory`.

---

## Gestión de Inventario de Tiendas

El sistema permite añadir o quitar ítems de las tiendas de forma dinámica (ej. después de completar una misión o desbloquear un nuevo nivel de gremio):
- `add_item_to_shop(shop_name, item_id)`
- `remove_item_from_shop(shop_name, item_id)`

---

## Ejemplos de Uso

### Abrir la Armería de la ciudad

```python
from Modules.ModulesManager.ShopManager import display_shop
from Modules.ModulesManager.ItemManager import get_item_manager

im = get_item_manager()
# Abre el menú visual de la tienda "WeaponShop"
display_shop("WeaponShop", player, im)
```

### Añadir un nuevo ítem a la venta tras un evento

```python
from Modules.ModulesManager.ShopManager import get_shop_manager

sm = get_shop_manager()
# Ahora la tienda mágica vende el ítem con ID 500
sm.add_item_to_shop("MagicShop", 500)
```
---

## Notas Técnicas

- Los precios de venta se obtienen directamente del campo `Gold_Cost` definido en `DataItems.json`.
- El sistema utiliza `interactive_menu_select` para una navegación fluida en consola.
