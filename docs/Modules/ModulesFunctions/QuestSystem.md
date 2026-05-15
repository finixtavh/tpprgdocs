# `QuestSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Misiones](#flujo)

---

## 1. Descripción General
El `QuestSystem.py` maneja las misiones clásicas del juego de rol (Quests) que el jugador puede aceptar para ganar Oro y Experiencia. Soporta actualmente misiones de recolección (`gather`) y de caza (`kill`). El sistema funciona pasivamente en el fondo, escuchando los eventos de combate y de inventario para actualizar los progresos hasta que el jugador reclama su recompensa en el Tablón de Misiones.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Lee el archivo `./Data/DataQuests.json` donde se declaran las plantillas de las misiones (Objetivos, Cantidades, Oro y EXP de recompensa).
- **Librerías TUI**: `rich.console`, `rich.panel`.
- **Inyecciones Condicionales**: `input_utils.interactive_menu_select` inyectada en el momento de renderizar el menú interactivo para evitar dependencias circulares al cargar el archivo.

---

## 3. Constantes y Variables Globales
- `quest_manager` (`QuestManager`): Instancia global inicializada automáticamente al final del archivo. Sirve como Singleton universal al que los otros sistemas llaman (ej: `MainGame` invoca `quest_manager.update_kill_quest(enemy.Name, player)` cada vez que el jugador asesina a alguien).

---

## 4. Clases y Estructuras de Datos

### `cQuest`
Representa una misión individual y su progreso actual.

#### Atributos Claves
- `self.type` (`str`): Tipo de misión (`"kill"` o `"gather"`).
- `self.target` (`str`): Identificador del objetivo (Ej: Nombre del enemigo a matar, Nombre del objeto a recolectar).
- `self.required_amount` (`int`): Cantidad requerida.
- `self.current_amount` (`int`): Progreso del jugador.
- `self.completed` (`bool`): Cambia a `True` cuando $Current \ge Required$.
- `self.reward_claimed` (`bool`): Bandera para asegurar que la recompensa se cobre una sola vez.

### `QuestManager`
Gestor central.

#### Colecciones Internas
- `self.quests` (`Dict`): Catálogo total de todas las misiones que existen en el JSON.
- `self.active_quests` (`Dict`): Las que el jugador tiene en su registro.
- `self.completed_quests` (`List[str]`): IDs de las misiones entregadas.

---

## 5. Funciones del Módulo (API)

- `load_quests(self)`: Parsea el JSON y alimenta el diccionario `self.quests`.
- `accept_quest(self, quest_id: str)`: Pasa una misión de "Disponible" a "Activa".
- `update_kill_quest(self, enemy_name: str, player)`: Hook de combate. Busca si el nombre del enemigo asesinado (sin case-sensitive) coincide con el `target` de alguna quest de tipo `"kill"`. Si es así, aumenta en 1 el contador y avisa en pantalla si se completó.
- `check_gather_quests(self, vInventory)`: Hook de inventario. A diferencia del contador acumulativo de `kill`, esta función recorre en tiempo real el inventario del jugador. Si la quest requiere 10 de "Madera" y el jugador tiene 10, la marca como completada. Si el jugador las tira, se des-completa (evaluación al vuelo).
- `claim_reward(self, quest_id, player, vInventory)`: Mutador final. Cobra el Oro y EXP dictaminado por la misión, llama a `player.level_up()`, y traslada la quest de `active` a `completed`.
- `open_quest_board(self, player, vInventory)`: Interfaz en consola. Muestra las misiones separando las Nuevas Disponibles de las Activas/Para Cobrar.

---

## 6. Flujo de Misiones

### Misión de Caza (`kill`)
1. El Jugador la acepta en el Tablón (`Matar 3 Goblins`).
2. Entra a una zona, pelea contra un Goblin y lo mata.
3. El motor de combate hace una llamada a `quest_manager.update_kill_quest("Goblin", player)`.
4. El sistema actualiza pasivamente: `Progreso: 1/3`.
5. Tras matar a 3, el motor de combate emite en texto verde que la misión está completa.
6. El jugador regresa al menú, abre el tablón, y le da a Cobrar.

### Misión de Recolección (`gather`)
1. El Jugador la acepta en el Tablón (`Traer 5 Hierba Curativa`).
2. Cierra el menú, y mediante el `GatheringSystem` o en combates consigue la hierba en su mochila.
3. El jugador vuelve a abrir el Tablón.
4. En la línea 121, `open_quest_board` ejecuta internamente `self.check_gather_quests(vInventory)`.
5. El sistema recorre la mochila entera, encuentra la hierba, actualiza su contador a $5/5$ e inmediatamente le avisa al jugador que ya puede cobrar la recompensa en ese mismo instante.
