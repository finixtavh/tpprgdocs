# `ItemBrowser.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo Visual e Interacción](#interacción)

---

## 1. Descripción General
`ItemBrowser.py` provee la "Enciclopedia" o "Bestiario de Ítems" del juego. Es un menú visual interactivo que permite al jugador navegar a través del catálogo completo de objetos registrados en el `ItemManager` (`DataItems.json`, `DataWeapons.json`, etc.). Implementa mecánicas de descubrimiento: los objetos que el jugador nunca ha tenido se muestran bloqueados (censurados con bloques "█"), incentivando la exploración.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Ninguno directamente, le delega la carga total a la instancia de `ItemManager`.
- **Librerías TUI**:
  - `rich.console`, `rich.panel`, `rich.table`, `rich.live` para el renderizado asíncrono a 60fps de interfaz dividida.
  - `sshkeyboard` para la escucha cruda de teclado.
- **Inyecciones Condicionales**: `input_utils.keyboard_listener_scope` para el manejo seguro del foco de la terminal. Utiliza las clases `cWeapon`, `cEquippableItems`, `cObject`, `cTool`, y `cMaterial` para instanciar el catálogo.

---

## 3. Constantes y Variables Globales
- `_TYPE_ICON` (`Dict`): Mapeo estético de íconos según la clase (Ej: `cWeapon` -> "⚔").
- `_TYPE_LABEL` (`Dict`): Traducción de la clase para interfaz de usuario.
- `_TYPE_COLOR` (`Dict`): Paleta de colores para cada familia de objetos (`bold red` para armas, `bold blue` para materiales).

---

## 4. Clases y Estructuras de Datos

### `ItemBrowser`
Clase envoltorio para precargar y ordenar el catálogo masivo antes de abrir la UI.

#### Atributos de Instancia
- `self.item_manager`: Instancia inyectada por el invocador.
- `self.all_items` (`List[object]`): Lista gigante donde se guardan las pre-instancias de TODOS los objetos del juego.

#### Métodos Principales
- `_load_all_items(self)`: Bucle anidado masivo que recorre `item_manager.items_data`. Intenta aislar las llaves (`STATIC_ID` o `MATERIAL_ID`) y manda a instanciar uno de cada objeto al vuelo usando la factoría. Luego los ordena por ID de menor a mayor.

---

## 5. Funciones del Módulo (API)

- `open_item_browser(player, item_manager)`
  - **Propósito**: Controlador principal que enciende la UI asíncrona.
- `_is_discovered(item, player) -> bool`
  - **Propósito**: Función auxiliar. Chequea si el ID estático del ítem pasado por parámetro se encuentra dentro del `Set` o `List` de `player.discovered_items`.
- `_obscure(text: str) -> str`
  - **Propósito**: Función auxiliar. Parsea una cadena de texto respetando los espacios pero sustituyendo todos los caracteres alfanuméricos por el bloque ANSI `█`. (Ej: "Espada Larga" $\rightarrow$ "██████ █████").

---

## 6. Flujo Visual e Interacción

Al llamar a `open_item_browser`:
1. El sistema carga una cuadrícula de dos columnas: **Lista (Izquierda)** y **Detalle (Derecha)**.
2. El bucle infinito `live.update()` comienza a renderizar a alta velocidad.
3. El teclado es atajado por `_on_press`.
   - **Navegación (`UP/DOWN`)**: Mueve el cursor, y si supera el borde inferior de la pantalla, incrementa el valor de `scroll` para mover la cámara hacia abajo ocultando los de arriba.
   - **Favoritos (`[F]`)**: Añade el objeto a `player.favorite_items`, lo que fuerza que el motor de renderizado lo dibuje arriba de todo en la lista junto con una estrella amarilla `★`.
   - **Búsqueda (`[S]`)**: Cambia el modo a `input_mode = "search"`. Atrapa todas las teclas tecleadas y las concatena en un String. La función interna `_get_filtered_items` filtra la lista original en tiempo real para hacer una búsqueda incremental sin case-sensitive. Al pulsar `[ENTER]`, congela el término y devuelve el control de las flechas.
4. Si un objeto se renderiza en **Detalle**, y no fue "Descubierto" (paso 2 de API), muestra únicamente:
   ```text
   [Icono] ██████ █████
   [?] Descubre este ítem para ver sus estadísticas.
   ```
