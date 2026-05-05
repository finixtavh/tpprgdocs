# `GuildMissions.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Tipos de Misiones](#tipos-de-misiones)
3. [Clase `GuildMission`](#clase-guildmission)
4. [Lógica de Resolución Autónoma](#lógica-de-resolución-autónoma)
5. [Bonus de Clase de Compañeros](#bonus-de-clase-de-compañeros)
6. [Interfaz de Usuario](#interfaz-de-usuario)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`GuildMissions.py` implementa el sistema de misiones externas donde el jugador puede enviar a sus **compañeros** (Companions) a realizar tareas para el gremio de forma autónoma. Mientras están en una misión, los compañeros no están disponibles para el grupo del jugador y su éxito depende de sus estadísticas y clase.

Las misiones avanzan a medida que el jugador realiza combates por su cuenta.

---

## Tipos de Misiones

| Tipo | Icono | Descripción | Efecto del Éxito |
|---|---|---|---|
| **Asalto** | ⚔ | Ataque a posición enemiga. | Conquista de territorio. |
| **Defensa** | 🛡 | Proteger territorio propio. | Evita pérdida de control. |
| **Sabotaje**| 🗡 | Infiltración en rivales. | Reduce poder militar enemigo. |
| **Recolección**| 📦 | Búsqueda de recursos. | Añade materiales a la Guild. |

---

## Clase `GuildMission`

Gestiona los datos y el estado de una misión en curso.

- `duration_combats`: Número de combates que debe realizar el jugador para que la misión termine.
- `assigned_companions`: Lista de IDs de compañeros enviados.
- `result`: Almacena si la misión fue un éxito o un fracaso tras completarse.

---

## Lógica de Resolución Autónoma

El éxito se calcula comparando el **Poder del Squad** contra el **Umbral de Dificultad**:

1. **Poder del Squad**: `suma de (HP_máx * 0.5 * (1 + bonus_clase))`.
2. **Dificultad**: `misión.difficulty * 12 + factor_aleatorio`.
3. **Resultado**: Si `squad_power >= dificultad`, la misión es un éxito.

---

## Bonus de Clase de Compañeros

Ciertas clases de compañeros son más efectivas en tipos específicos de misiones:

- **Tanque**: +40% en misiones de **Defensa**.
- **Explorador**: +40% en **Recolección**, +30% en **Sabotaje**.
- **Daño**: +30% en **Asalto**.
- **Mago**: Bonus equilibrado en Asalto (+20%) y Sabotaje (+20%).

---

## Interfaz de Usuario

### `open_guild_missions_menu(guild, player, guild_manager)`
Una interfaz interactiva que permite:
- Ver misiones disponibles generadas aleatoriamente.
- Consultar recompensas (Materiales, Puntos de Skill).
- Seleccionar y enviar compañeros.
- Muestra el estado de los slots de misión (ampliables mediante el árbol de habilidades).

---

## Ejemplos de Uso

### Consultar finalización de misiones
El motor principal del juego llama a esta función tras cada combate:

```python
from Modules.ModulesFunctions.GuildMissions import check_mission_completion

# Recorrer misiones activas de la guild del jugador
for m_data in player_guild.active_missions:
    m = GuildMission.from_dict(m_data)
    res = check_mission_completion(m, player.combat_counter, player.companion, player_guild, gm)
    if res:
        # Procesar fin de misión...
```
---

## Notas y Referencias

- Los compañeros en misión pasan al estado `on_mission`.
- Al regresar, pasan al estado `inactive`, lo que significa que el jugador debe volver a añadirlos manualmente a su grupo principal si desea viajar con ellos.
- Las misiones de **Asalto** fallidas pueden provocar la pérdida de soldados de la población de la Guild.
