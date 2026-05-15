# `GuildSkillTree.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Estructura del Árbol (Ramas)](#ramas)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo de Interfaz Visual](#flujo-visual)

---

## 1. Descripción General
`GuildSkillTree.py` gestiona un árbol de habilidades exclusivo para el ecosistema geopolítico (Guilds). Funciona de forma paralela y separada al `SkillTreeSystem.py` del Jugador. Utiliza una moneda diferente (`guild_skill_points`) obtenida a través de misiones de clan, y las bonificaciones otorgadas (como mejor comercio, poder militar o mayor ganancia de moral) afectan a todos los cálculos macro del `GuildManager`.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: A diferencia del jugador que lee un JSON para su árbol, el árbol de Guild está _hardcodeado_ (programado directamente en este archivo) bajo la constante `GUILD_SKILL_TREE`.
- **Librerías TUI**: `rich` (`Table`, `Panel`, `Text`, `Live`) y `sshkeyboard` para dibujar y navegar la interfaz gráfica.
- **Inyecciones Condicionales**: `input_utils` (`keyboard_listener_scope`).

---

## 3. Constantes y Variables Globales
- `GUILD_SKILL_TREE` (`Dict[str, dict]`): El diccionario base que define cada nodo.
- `BRANCH_ORDER` (`Dict[str, list]`): Agrupación y orden visual de los nodos en 3 grandes categorías (`Militar`, `Economía`, `Diplomacia`).
- `BRANCH_ICONS` (`Dict[str, str]`): Iconografía para las pestañas de UI.

---

## 4. Estructura del Árbol (Ramas)

Cada habilidad es un diccionario con `id`, `cost`, `requires`, y un objeto `effect` especial que el `GuildSystem` consultará en sus matemáticas.

### ⚔ Rama Militar
Mejora el desempeño bélico de la guild en `GuildSystem._simulate_wars()` y abre slots de misiones autónomas.
- **Nodos Clave**: `mil_vanguard` (+1 Slot Misiones), `mil_assault` (+25% bonus asalto), `mil_professional` (+40% poder militar base), `mil_conscription` (+2% ganancia pasiva poblacional).

### 🌾 Rama Economía
Impacta las matemáticas del ciclo económico (`SIMULATION_INTERVAL`).
- **Nodos Clave**: `eco_logistics` (+20% producción), `eco_granaries` (x2 eficiencia de comida, mitigando crisis de hambruna), `eco_trade` (Desbloquea propuesta de intercambios con NPCs).

### 🏛 Rama Diplomacia
Influye en el motor de reputación y las decisiones de la IA en eventos.
- **Nodos Clave**: `dip_ambassador` (+15% reputación), `dip_espionage` (permite espiar a otros clanes), `dip_shadows` (+50% bonus para misiones de sabotaje).

---

## 5. Funciones del Módulo (API)

### Matemática e Intérprete de Efectos
- `can_unlock(skill_id, guild) -> Tuple[bool, str]`: Función que pre-valida si la guild tiene los `skill_points` suficientes y si ya completó todos los IDs indicados en `requires`. Retorna booleano y un *string* descriptivo en caso de error.
- `unlock_skill(skill_id, guild) -> Tuple[bool, str]`: Resta el costo, pushea el ID a la lista `guild.unlocked_skills`.
- **Getters Especiales**: En lugar de mutar propiedades como el árbol del jugador, aquí el sistema expone getters que recalculan los bonos al vuelo leyendo el array `guild.unlocked_skills`:
  - `get_effective_mission_slots(guild) -> int`: Lee si existe `mil_vanguard`.
  - `get_war_bonus(guild) -> float`: Suma `mil_tactics` y `mil_professional`.
  - `get_resource_bonus(guild) -> float`
  - `get_food_efficiency(guild) -> float`

### Interfaz Principal
- `open_guild_skill_tree(guild, console)`
  - Abre el panel Live.

---

## 6. Flujo de Interfaz Visual

1. La interfaz se divide en Pestañas (Tab View) utilizando las categorías de `BRANCH_ORDER`.
2. El usuario usa `IZQUIERDA` / `DERECHA` para rotar entre las pestañas (Militar -> Economía -> Diplomacia).
3. Con `ARRIBA` / `ABAJO`, selecciona la habilidad en la lista de la pestaña activa.
4. El renderizador asíncrono (`_render`) dibuja un Icono de Status:
   - `✓` Verde oscuro: Ya comprado.
   - `▶` Cian brillante: Seleccionado.
   - Si no está seleccionado pero se puede comprar: Blanco "Disponible".
   - Si falta árbol o puntos: Gris oscuro "Requiere...".
5. Al pulsar `ENTER`, invoca `unlock_skill` y recarga la pantalla.
