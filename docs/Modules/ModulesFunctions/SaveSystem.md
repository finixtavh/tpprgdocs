# `SaveSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Estructura de Archivos](#estructura-de-archivos)
3. [Proceso de Serialización](#proceso-de-serialización)
4. [Gestión de Slots (1, 2 y 3)](#gestión-de-slots-1-2-y-3)
5. [Restauración de Estado](#restauración-de-estado)
6. [Integración con Módulos](#integración-con-módulos)
7. [Interfaz de Usuario](#interfaz-de-usuario)
8. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`SaveSystem.py` es el componente encargado de la persistencia de datos. Permite guardar y cargar el estado completo del juego en archivos JSON. El sistema es robusto y cubre no solo las estadísticas del jugador, sino también inventarios, encantamientos, progreso de la base, misiones, reputaciones y el estado global de la simulación de gremios.

---

## Estructura de Archivos

Las partidas se almacenan en `Data/Saves/` bajo los nombres:
- `Slot1.json`
- `Slot2.json`
- `Slot3.json`

Cada archivo contiene un objeto JSON con metadatos de versión y timestamp, seguido de secciones para cada sistema del juego.

---

## Proceso de Serialización

El sistema descompone los objetos complejos de Python en diccionarios serializables:
- **Items**: Se guardan por su ID, categoría y lista de encantamientos (ID y nivel).
- **Jugador**: Se guardan todos los atributos numéricos, sets de ítems descubiertos y listas de favoritos.
- **Compañeros**: Incluye identidad (nombre generado, género), estado vital y equipo actual.
- **Inventario**: Lista de slots con ID de ítem y cantidad.

---

## Gestión de Slots (1, 2 y 3)

### `get_slot_info(slot) -> dict`
Permite obtener una vista previa de la partida sin cargarla completamente. Retorna:
- Nombre del personaje.
- Nivel.
- Zona actual.
- Fecha y hora del guardado.

---

## Restauración de Estado

Al cargar una partida, el sistema sigue un orden jerárquico para reconstruir los objetos:
1. Reconstruye el objeto `player` y sus stats.
2. Utiliza el `ItemManager` para recrear las instancias de ítems en el inventario y equipo.
3. Re-instancia a los compañeros y les asigna su equipo guardado.
4. Restaura los Singleton de los gestores (`MasteryManager`, `BaseManager`, `QuestManager`, `GuildManager`).

---

## Integración con Módulos

El `SaveSystem` actúa como un punto de unión para todos los sistemas funcionales:
- **GuildSystem**: Guarda el mapa de territorios y relaciones geopolíticas.
- **QuestSystem**: Guarda misiones activas y el historial de completadas.
- **GatheringMasterySystem**: Guarda los niveles de habilidades de recolección (Minería, Botánica, etc.).

---

## Interfaz de Usuario

### `save_load_menu(...)`
Muestra un menú interactivo que permite elegir entre Guardar o Cargar.
- En el modo **Guardar**, muestra qué slots están ocupados para prevenir sobrescrituras accidentales.
- En el modo **Cargar**, muestra la información detallada de cada slot (Nombre, Nv, Zona).

---

## Ejemplos de Uso

### Guardar partida automáticamente (Autosave)

```python
from Modules.ModulesFunctions.SaveSystem import save_to_slot

# Guardar en el Slot 1 (típicamente usado para autosave)
save_to_slot(1, player, inventory, "Ciudad Principal")
```

### Cargar al iniciar el juego

```python
from Modules.ModulesFunctions.SaveSystem import load_from_slot

game_state = load_from_slot(1, player, inventory, item_manager)
if game_state:
    print(f"Bienvenido de nuevo, {player.name}. Estás en {game_state['chosen_area']}")
```
