# `ItemBrowser.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `ItemBrowser`](#clase-itembrowser)
3. [Mecánica de Descubrimiento](#mecánica-de-descubrimiento)
4. [Interfaz de Usuario](#interfaz-de-usuario)
5. [Controles](#controles)
6. [Notas Técnicas](#notas-técnicas)

---

## Descripción General

`ItemBrowser.py` implementa la **Enciclopedia de Ítems** (o Bestiario/Catálogo) del juego. Es una herramienta de consulta que permite al jugador ver todos los objetos existentes en la base de datos del juego, filtrarlos y marcar favoritos.

---

## Clase `ItemBrowser`

Se encarga de indexar todos los ítems disponibles en el `ItemManager`.

- `_load_all_items()`: Escanea todas las categorías de ítems (Armas, Armaduras, Materiales, etc.) y crea instancias temporales para mostrar su información.
- Los ítems se ordenan automáticamente por su ID numérico.

---

## Mecánica de Descubrimiento

El navegador respeta la progresión del jugador:
- Si un ítem **ha sido descubierto** (está en `player.discovered_items`), se muestra su nombre, descripción y estadísticas completas.
- Si el ítem **es desconocido**, su nombre y descripción aparecen censurados (bloques `█`) y sus estadísticas están ocultas.

---

## Interfaz de Usuario

### `open_item_browser(player, item_manager)`
La interfaz utiliza un diseño de doble panel:
1. **Panel Izquierdo (Lista)**: Muestra los ítems con iconos representativos de su tipo. Los favoritos aparecen con una estrella dorada `★` al inicio.
2. **Panel Derecho (Detalle)**: Muestra la información técnica (Daño, Defensa, Valor en oro, Descripción narrativa).

---

## Controles

| Tecla | Acción |
|---|---|
| **Flechas ↑↓** | Navegar por la lista. |
| **S** | Activar modo búsqueda (filtrar por nombre). |
| **F** | Marcar/Desmarcar como favorito. |
| **ESC** | Salir del navegador. |
| **Backspace** | Borrar caracteres en el modo búsqueda. |

---

## Notas Técnicas

- **Filtros**: La búsqueda es insensible a mayúsculas/minúsculas.
- **Favoritos**: Los ítems marcados como favoritos siempre aparecen al principio de la lista, independientemente de su ID.
- **Iconografía**:
    - ⚔ : Armas
    - 🛡 : Equipamiento/Armaduras
    - 🧪 : Objetos consumibles
    - ⛏ : Herramientas
    - 🪨 : Materiales
