# `GuildSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Estructuras de Datos](#estructuras-de-datos)
    - [GuildResource](#guildresource)
    - [Territory](#territory)
3. [Clase Principal: `Guild`](#clase-principal-guild)
4. [Gestor: `GuildManager`](#gestor-guildmanager)
5. [Sistema de Economía y Producción](#sistema-de-economía-y-producción)
6. [Sistema Militar y Combate](#sistema-militar-y-combate)
7. [IA de Gremios (NPC)](#ia-de-gremios-npc)
8. [Eventos y Desastres](#eventos-y-desastres)
9. [Ciclo de Juego (Diagrama)](#ciclo-de-juego-diagrama)
10. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`GuildSystem.py` es el motor central que gestiona la geopolítica, economía y simulación de los gremios (Guilds) en TPPRPG. Permite la existencia de múltiples facciones con intereses propios, territorios que producen recursos y ejércitos que pueden conquistar nuevas tierras.

**Características principales:**
- **IA Estratégica**: Los gremios NPC operan bajo 4 estados: *Survival, Expansion, Economic y Balanced*.
- **Sistema de Impuestos (V4)**: El jugador puede ajustar la tasa impositiva (1-50%) cada 5 peleas.
- **Tensión Mundial**: Un indicador global que aumenta con las declaraciones de guerra.
- **Espionaje y Sabotaje**: Permite ver stats de enemigos y realizar ataques encubiertos.

---

## Estructuras de Datos

### `GuildResource`
Representa un tipo de recurso (Madera, Piedra, Oro, etc.) dentro del inventario de un gremio.
- Atributos: `resource_id`, `name`, `icon`, `rarity`, `amount`.

### `Territory`
Representa una zona en el mapa mundial.
- Atributos: `owner`, `resource_output`, `defense_value`, `loyalty`, `garrison`.
- Los territorios pueden rebelarse si la lealtad (`loyalty`) cae a 0 debido a falta de guarnición.

---

## Clase Principal: `Guild`

Define el estado y comportamiento de un gremio.

### Atributos Clave
- **Estadísticas**: `military_power`, `population`, `moral`, `aggressiveness`.
- **Economía**: `tax_rate`, `tax_gold_pool`, `resources` (Dict de `GuildResource`).
- **Militar**: `soldiers` (Infantry, Archers, Cavalry, Siege), `attack_rating`, `defense_rating`.
- **Infraestructura**: `buildings`, `unlocked_skills`.
- **Global**: `at_war_with`, `relations`.

### Métodos Destacados
- `recruit_soldiers(type, count)`: Convierte recursos y población en tropas.
- `apply_taxes()`: Recauda oro basado en la población y la tasa de impuestos. Afecta negativamente a la moral si los impuestos son altos (>10%).
- `grow_population()`: Calcula el crecimiento poblacional basado en el suministro de comida.
- `clamp_resources()`: Limita los recursos según la capacidad de los almacenes del gremio.

---

## Gestor: `GuildManager`

El Singleton `get_guild_manager()` coordina la simulación global.

- `generate_random_guilds()`: Crea la población inicial de gremios NPC.
- `initialize_world_state()`: Configura dinámicamente la situación geopolítica al iniciar una nueva partida leyendo el `config.json`, determinando la cantidad de conflictos iniciales (`AMOUNT_OF_STARTING_WARS`) y ajustes diplomáticos (`AMOUNT_OF_STARTING_EVENTS`).
- `generate_initial_player_events()`: Al fundar o unirse a un gremio, pre-genera de manera procedural una serie de eventos interactivos iniciales (Bienvenida, Mercader, Diplomacia), eliminando la dependencia de un paso de tiempo global en las etapas iniciales de la simulación.
- `tick()`: El corazón de la simulación. Se ejecuta periódicamente para procesar producción, IA, eventos y desastres.
- `_process_disasters()`: Aplica efectos climáticos destructivos (ej. terremotos) a los gremios, con probabilidad reducida si poseen un "Puesto de Mantenimiento".

---

## Sistema Militar y Combate

El combate entre gremios (`_attack_territory`) es una resolución matemática que considera:
1. **Poder Base**: El `military_power` del gremio.
2. **Composición de Tropas**: Cada tipo de tropa tiene un valor y hay un sistema de "counters" (ej. Caballería vence Arqueros).
3. **Bonificaciones**: Árbol de habilidades, edificios (Murallas, Cuartel) y moral.
4. **Debuffs de Impuestos**: Si la tasa es >10%, se aplican penalizaciones severas al Daño y la Defensa del jugador.
5. **Clima**: Los modificadores del `WeatherSystem` afectan el resultado final.

---

## IA de Gremios (NPC)

La IA toma decisiones en cada `tick`:
1. **Sobrevivir**: Si falta comida, ataca territorios con producción agrícola.
2. **Expandir**: Si tiene moral y poder alto, intenta conquistar territorios neutrales o enemigos.
3. **Diplomacia**: Forma coaliciones contra gremios dominantes o declara guerras si la relación es baja.
4. **Construir**: Gasta excedentes de recursos en edificios estratégicos.

---

## Ciclo de Juego (Diagrama)

```mermaid
graph TD
    A[Inicio del Tick] --> B[Producción de Recursos]
    B --> C[Crecimiento de Población]
    C --> D[Recaudación de Impuestos (Cada 5 Peleas)]
    D --> E[Decisión de la IA (Estratégica)]
    E --> F{¿Atacar o Sabotear?}
    F -- Sí --> G[Resolución de Combate / Sabotaje]
    F -- No --> H[Construcción/Reclutamiento/Trade]
    G --> I[Aumento de Tensión Mundial]
    H --> I
    I --> J[Eventos y Desastres]
    J --> K[Fin del Tick]
```

---

## Notas y Referencias

- **Gobernadores**: Asignar un compañero como gobernador a un territorio aumenta su producción en un +20%.
- **Mantenimiento**: El edificio `mantenimiento` reduce en un 75% la probabilidad de perder edificios por desastres.
- **Relaciones**: La reputación del jugador con cada gremio afecta los precios de comercio y la probabilidad de ser atacado.
