# `InventoryMenu.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Interfaz de Doble Panel](#interfaz-de-doble-panel)
3. [Mecánicas de Descubrimiento](#mecánicas-de-descubrimiento)
4. [Visualización de Atributos](#visualización-de-atributos)
    - [Armas y Equipamiento](#armas-y-equipamiento)
    - [Consumibles y Herramientas](#consumibles-y-herramientas)
    - [Materiales](#materiales)
5. [Controles y Acciones](#controles-y-acciones)
6. [Integración con `EquipmentManager`](#integración-con-equipmentmanager)

---

## Descripción General

`InventoryMenu.py` proporciona una interfaz moderna y fluida para que el jugador gestione sus pertenencias. Utiliza un diseño de doble panel (lista a la izquierda, detalle a la derecha) y permite el uso o equipamiento instantáneo de objetos mediante atajos de teclado.

---

## Interfaz de Doble Panel

### Panel de Lista (Izquierda)
Muestra una lista numerada de todos los ítems en el inventario actual.
- **Iconos**: Representación visual del tipo de ítem (⚔ para armas, 🛡 para armaduras, etc.).
- **Colores**: Los nombres cambian de color según la rareza del ítem.
- **Scroll Inteligente**: Soporta listas largas mediante desplazamiento automático.

### Panel de Detalle (Derecha)
Se actualiza en tiempo real al navegar por la lista. Muestra:
- Nombre y tipo de ítem.
- Descripción narrativa (o censurada si no se ha descubierto).
- Atributos técnicos representados con barras visuales `██░░`.

---

## Mecánicas de Descubrimiento

El menú respeta el sistema de exploración del juego. Si un ítem está en el inventario pero su ID no figura en `player.discovered_items`:
- El nombre y descripción aparecerán censurados con bloques `████`.
- Los atributos técnicos se ocultarán.
- Se mostrará una advertencia de "Ítem no descubierto".

---

## Visualización de Atributos

### Armas y Equipamiento
Muestra barras de daño y defensa para los 5 tipos elementales. Indica también bonus de HP, robo de vida y daño extra.

### Consumibles y Herramientas
- **Pociones**: Muestra cuánto HP/MP restauran.
- **Herramientas**: Muestra la eficiencia y los tipos de materiales que pueden recolectar.

### Materiales
Muestra la dureza del recurso, la rareza y la habilidad de recolección requerida (Minería, Tala, etc.).

---

## Controles y Acciones

| Tecla | Acción |
|---|---|
| **↑ / ↓** | Navegar por la lista de ítems. |
| **E** | Equipar arma, armadura o herramienta. |
| **U** | Usar objeto consumible (Pociones, pergaminos). |
| **I** | Abrir pantalla de información completa (Pantalla completa). |
| **Q / Esc** | Salir del inventario. |

---

## Integración con `EquipmentManager`

El menú de inventario no gestiona el equipamiento por sí solo. Al presionar **E**, delega la tarea al `EquipmentManager`, que se encarga de calcular los cambios de estadísticas y gestionar los slots (incluyendo la selección de slots para anillos).

---

## Notas Técnicas

- Renderizado ultra-rápido (30 FPS) mediante `rich.live`.
- Soporte para visualización de encantamientos (inyectado desde el `EnchantmentSystem`).
- El estado de "Equipado" se indica visualmente con una marca `✓` verde en el panel de detalle.
