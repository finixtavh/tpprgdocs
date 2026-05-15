# `GuildMissions.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Resolución Autónoma](#flujo-de-resolución)

---

## 1. Descripción General
`GuildMissions.py` introduce una mecánica "Idle" / "Dispatch" dentro del juego. Permite a los líderes de un clan (Guild) enviar a los Compañeros (`cCompanion`) contratados a realizar misiones asíncronas en el trasfondo. Estas misiones toman un número específico de *combates del jugador* para completarse, y si los compañeros tienen las estadísticas suficientes para sobrevivir, regresan con recursos o conquistan territorios sin necesidad de que el jugador pelee manualmente.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Ninguno directo. Recibe definiciones geográficas importando indirectamente `GuildManager`.
- **Librerías TUI**: Uso extensivo de `rich.console`, `rich.table`, `rich.live` para mostrar la pantalla de selección de misiones, y `sshkeyboard` para la navegación con flechas en la UI.
- **Inyecciones Condicionales**: Importa `get_effective_mission_slots` desde `GuildSkillTree.py` para calcular dinámicamente si el clan tiene capacidad para lanzar otra misión paralela.

---

## 3. Constantes y Variables Globales
- `MISSION_TYPES` (`Dict`): Catálogo estático con 4 misiones posibles.
  - **Asalto**: Combate contra un territorio enemigo para conquistarlo. (Dificultad Base: 50).
  - **Defensa**: Frenar ataque enemigo a un territorio propio. (Base: 35).
  - **Sabotaje**: Atacar las reservas militares de otra guild. (Base: 45).
  - **Recolección**: Buscar materias primas. (Base: 20).
- `CLASS_MISSION_BONUS` (`Dict`): Matriz de afinidad. Define que enviar a un compañero clase "Tanque" a una Defensa le da `+40%` de fuerza, pero en Sabotaje le da `0%`.

---

## 4. Clases y Estructuras de Datos

### `GuildMission`
Estructura de datos que encapsula el contrato de una misión.

#### Atributos de Instancia (Relevantes)
- `self.mission_id` / `self.mission_type`: Identificadores lógicos.
- `self.difficulty` (`int`): Multiplicador matemático requerido para pasar el check (de 1 a 10).
- `self.duration_combats` (`int`): Condición de victoria. Cuántos monstruos debe matar el Jugador (en su juego normal) antes de que el servidor considere que la misión expiró.
- `self.reward_resources` / `self.reward_skill_pts`: Qué gana el clan si hay éxito.
- `self.assigned_companions` (`List[str]`): Lista con los IDs de los NPC mandados al trabajo.
- `self.started_at_combat` (`int`): Fotografía del contador de combates del jugador (`player.combat_counter`) en el instante en que se presiona Enter.

#### Métodos Principales
- `to_dict(self)` / `from_dict(cls, data)`: Serialización plana.

---

## 5. Funciones del Módulo (API)

- `generate_missions_for_guild(guild, guild_manager) -> List[GuildMission]`
  - **Propósito**: Observa el estado actual del mundo (Ej: si la guild tiene territorios enemigos cerca, o si hay neutrales) y crea proceduralmente 4 misiones para ofrecerlas en el tablón.
- `open_guild_missions_menu(guild, player, guild_manager, console)`
  - **Propósito**: Panel interactivo. Lista las 4 misiones creadas. Permite seleccionar una y despachar automáticamente a un compañero de la reserva marcando su `mission_status = "on_mission"`.
- `check_mission_completion(mission, current_combat, companions, guild, guild_manager)`
  - **Propósito**: Hook de chequeo pasivo que debe inyectarse en el ciclo principal del juego. Revisa si el jugador ya peleó lo suficiente (`current_combat >= started_at_combat + duration_combats`). Si es verdadero, detona el RNG matemático de la resolución.

---

## 6. Flujo de Resolución Autónoma

La lógica principal se encuentra en `resolve_autonomous_mission`:

1. El sistema recupera el escuadrón enviado (1 o más NPCs).
2. Calcula su poder base ($HP_{max} * 0.5$).
3. Aplica los bonos multiplicadores cruzando la matriz `CLASS_MISSION_BONUS` usando la clase del NPC contra el tipo de misión.
4. Si la Guild tiene bonificaciones pasivas desbloqueadas (Ej: `mil_assault` en el Árbol de Habilidades del Clan), el poder se infla un $25\%$.
5. Calcula el `difficulty_threshold`: $(dificultad \times 12) \pm Random(-10, +20)$.
6. Compara: Si $Poder > Umbral$, el jugador gana y recibe el Botín + Moral.
7. Si pierde, recibe daño al clan, baja moral, e incluso suscribe una penalización adicional si era Asalto o Sabotaje, donde la Guild sufre bajas definitivas poblacionales/militares. Los aliados regresan con estatus `"inactive"`.
