# `QuestSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `cQuest`](#clase-cquest)
3. [Tipos de Misiones](#tipos-de-misiones)
4. [Gestor de Misiones (`QuestManager`)](#gestor-de-misiones-questmanager)
5. [El Tablón de Misiones](#el-tablón-de-misiones)
6. [Ciclo de Vida de una Misión](#ciclo-de-vida-de-una-misión)

---

## Descripción General

`QuestSystem.py` implementa el sistema de misiones (quests) del juego. Permite a los jugadores aceptar tareas, realizar un seguimiento de su progreso y reclamar recompensas de oro y experiencia.

---

## Clase `cQuest`

Representa una misión individual.
- **quest_id**: Identificador único (leído de `DataQuests.json`).
- **target**: El objetivo de la misión (nombre de enemigo o ítem).
- **required_amount**: Cantidad necesaria para completar el objetivo.
- **current_amount**: Cantidad acumulada por el jugador.

---

## Tipos de Misiones

El sistema soporta actualmente dos tipos de objetivos:
- **`kill`**: El jugador debe derrotar a una cantidad específica de un enemigo determinado. El progreso se actualiza automáticamente al finalizar un combate.
- **`gather`**: El jugador debe recolectar ítems. El sistema verifica el inventario del jugador cada vez que abre el tablón de misiones para actualizar el progreso.

---

## Gestor de Misiones (`QuestManager`)

Centraliza la lógica de todas las misiones.
- **`load_quests()`**: Carga el catálogo base desde `Data/DataQuests.json`.
- **`accept_quest(id)`**: Mueve una misión del catálogo a la lista de `active_quests`.
- **`update_kill_quest()`**: Inyectada en el loop de combate para detectar muertes de enemigos objetivo.
- **`check_gather_quests()`**: Escanea el inventario en busca de materiales requeridos.
- **`claim_reward(id)`**: Otorga el oro/exp y mueve la misión a `completed_quests` (evitando que se repita si no es una misión diaria).

---

## El Tablón de Misiones

Interfaz interactiva (`open_quest_board`) que organiza las misiones en dos secciones:
1. **Activas**: Muestra el progreso actual (ej. `2/5`) y permite cobrar la recompensa si están marcadas como completadas.
2. **Nuevas**: Muestra misiones disponibles con el detalle de sus recompensas.

---

## Ciclo de Vida de una Misión

1. **Catálogo**: Definida en JSON.
2. **Aceptada**: El jugador la selecciona en el tablón.
3. **En Progreso**: El jugador realiza acciones en el mundo.
4. **Completada**: `current_amount >= required_amount`. Se muestra un aviso visual parpadeante.
5. **Reclamada**: El jugador vuelve al tablón y cobra su recompensa.

---

## Notas Técnicas

- El estado de las misiones se guarda en el archivo de guardado del jugador, separando las misiones activas de las ya finalizadas para evitar bugs de duplicación de recompensas.
- Soporta inyección de misiones mediante mods a través de `mod_api.quest_injections`.
