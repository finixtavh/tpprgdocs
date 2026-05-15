# `GuildSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Mecánicas Geopolíticas (Guerras y Eventos)](#mecanicas)

---

## 1. Descripción General
`GuildSystem.py` es el motor masivo de Geopolítica de TPPRPG. Simula un mundo vivo donde docenas de facciones (Guilds) compiten por el control de territorios en un mapa 2D, gestionan recursos (`GuildResources`), reclutan ejércitos (`soldiers`, `archers`, `cavalry`) y se declaran la guerra mutuamente usando inteligencia artificial simplificada. El jugador puede crear su propia Guild o unirse a una existente, participando activamente en la diplomacia y gestión (Moral, Relaciones y Comida) mediante decisiones en eventos procedimentales.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**:
  - `./Data/GuildResources.json`: Fórmulas y rarezas de ítems de clan (Madera, Hierro, Xar).
  - `./Data/Guilds.json`: Nombres base y arquetipos de clanes NPC.
  - `./Data/ZoneGuilds.json`: Topología del mapa, coordenadas `[X,Y]` y multiplicadores de defensa/población de cada zona.
  - `config.json` (`GUILDS_PARAMS`): Define la agresión de IA, penalties por guerra sorpresa y bonos base.
- **Inyecciones**: `WeatherSystem` (`apply_weather_combat_mods`) inyectado como bloqueador para penalizar matemáticamente las ofensivas masivas en territorios con climas adversos (Tormentas, Niebla).

---

## 3. Constantes y Variables Globales
- `SIMULATION_INTERVAL` (`int`): Segundos en tiempo real entre cada "Tick" económico mundial (por defecto 5s).
- `RARITY_BASE_CAPS` (`Dict`): Capacidad límite de almacenamiento base según rareza.
- `GUILD_BUILDINGS` (`Dict`): Diccionario anidado de los 11 edificios construibles en el clan (Cuartel, Granero, Mercado...) con costos en recursos.
- `GUILD_EVENTS_POOL` (`List[Dict]`): Plantillas de eventos procedimentales (Plagas, Espías, Peticiones de ayuda).

---

## 4. Clases y Estructuras de Datos

### `GuildResource`
Clase contenedora (`resource_id`, `amount`). Facilita la serialización JSON.

### `Territory`
Representa un nodo geográfico en disputa.
- **Atributos Clave**: `resource_output` (qué produce), `owner` (dueño actual), `garrison` (tropas estáticas apostadas), `loyalty` (si baja a 0 puede rebelarse), `position` (tupla X,Y).

### `Guild`
Entidad principal (Facciones NPC o del Jugador).
- **Atributos Económicos**: `resources`, `population`, `population_capacity`, `moral`.
- **Atributos Militares**: `soldiers`, `military_power`, `buildings` (Lista de IDs).
- **Atributos Diplomáticos**: `relations` (Dict mapeando ID Facción -> Relación -100 a +100), `at_war_with` (List de IDs), `event_queue` (Lista de eventos esperando decisión del jugador).
- **Métodos**: `add_resource`, `consume_resources`, `recruit_soldiers`, `calculate_military_power`.

### `GuildManager`
Singleton que simula el mundo.
- **Atributos**: `guilds` (Diccionario gigante), `territories`, `player_guild_id`, `last_simulation_tick`.
- **Métodos Internos Principales**:
  - `simulate_world()`: Bucle principal atado al tiempo. Ejecuta el consumo de comida general, producción, y da paso a la Inteligencia Artificial.
  - `_simulate_npc_decisions(guild)`: Motor IA. Analiza las relaciones de los NPC. Si odian a un vecino (`rel < -30`), y tienen 20% más fuerza militar, les declaran la guerra. Construyen edificios si sobra comida/piedra.
  - `_simulate_wars()`: Calcula el desgaste de guerra. Los clanes en guerra activa pierden tropas, población y recursos en cada tick hasta que la fuerza de un bando cae críticamente y se pide tregua.
  - `push_event_to_player()`: Lanza un evento extraído de `GUILD_EVENTS_POOL` a la cola del jugador para que lo responda en el menú M.

---

## 5. Funciones del Módulo (API)

- `get_guild_manager() -> GuildManager`: Singleton Global.
- `reset_guild_manager() -> GuildManager`: Borra la instancia de memoria y reinicia. Útil al cambiar de Partida guardada.
- `initialize_world_state()`: Función de Generación. Parsea los JSON, coloca a los NPC aleatoriamente en el mapa y precalcula los balances iníciales.
- `generate_initial_player_events(guild_id)`: Empuja los primeros 3 eventos predefinidos (Discurso, Bienvenida, Finanzas) a la cola del jugador recién ungido como Líder.

---

## 6. Mecánicas Geopolíticas (Guerras y Eventos)

### Ciclo Económico
Cada `SIMULATION_INTERVAL` (Tick), todos los clanes ganan recursos basados en los `Territory` que poseen y su nivel de `population`. Seguidamente, pagan "Mantenimiento" en Comida. Si la comida llega a 0, la `moral` entra en picada. Si la moral baja a críticos, empiezan las revueltas poblacionales y pérdida militar.

### Sistema de Eventos
Las notificaciones procedimentales se acumulan en `player_guild.event_queue`. Al abrir el menú, el jugador debe decidir entre 2 o 3 opciones. Las respuestas ejecutan modificadores estáticos de la plantilla (ej. `{"comida": -30, "rel_src": +15}`). Los "Targets" dinámicos (quién es el aliado que pide ayuda) se escogen aleatoriamente entre las facciones vecinas o aliadas, por lo que el texto es adaptado (`"Tus guardias capturaron a un espía de Gremio del Fuego"`).
