# `GuildSkillTree.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Estructura del Árbol](#estructura-del-árbol)
    - [Rama Militar](#rama-militar-⚔)
    - [Rama Economía](#rama-economía-🌾)
    - [Rama Diplomacia](#rama-diplomacia-🏛)
3. [Lógica de Desbloqueo](#lógica-de-desbloqueo)
4. [Funciones de Cálculo de Bonus](#funciones-de-cálculo-de-bonus)
5. [Interfaz de Usuario](#interfaz-de-usuario)
6. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`GuildSkillTree.py` implementa el sistema de progresión tecnológica y política de los gremios. A diferencia del árbol de habilidades del jugador, este sistema utiliza `guild_skill_points` y afecta a toda la organización (producción, guerra, diplomacia).

Cada gremio tiene su propio progreso independiente en el árbol.

---

## Estructura del Árbol

El árbol se divide en tres ramas principales, cada una con una progresión jerárquica (requiere haber desbloqueado nodos previos).

### Rama Militar (⚔)
Enfocada en el poder de combate, slots de misiones y bonos de asedio.
- **Nodos Clave**: `Tácticas Básicas`, `Vanguardia` (+1 slot misión), `Ejército Profesional` (+40% poder), `Maestría de Asedio`.

### Rama Economía (🌾)
Enfocada en la producción de recursos, eficiencia alimentaria y comercio.
- **Nodos Clave**: `Logística` (+20% producción), `Red Comercial` (Desbloquea comercio entre gremios), `Hegemonía Económica`, `Rutas de la Seda`.

### Rama Diplomacia (🏛)
Enfocada en reputación, espionaje y alianzas.
- **Nodos Clave**: `Alianzas` (Permite alianzas formales), `Espionaje` (Ver stats de enemigos), `Sombras Alargadas` (+50% Sabotaje), `Hegemonía Política`.

---

## Lógica de Desbloqueo

El sistema verifica tres condiciones para permitir el desbloqueo de un nodo:
1. **No poseído**: El gremio no debe tener ya la habilidad.
2. **Requisitos**: Todos los nodos indicados en `requires` deben estar desbloqueados.
3. **Costo**: El gremio debe tener suficientes `skill_points`.

- `can_unlock(skill_id, guild) -> (bool, reason)`: Realiza las verificaciones anteriores.
- `unlock_skill(skill_id, guild) -> (bool, message)`: Ejecuta el desbloqueo, resta los puntos y añade el ID a `guild.unlocked_skills`.

---

## Funciones de Cálculo de Bonus

Estas funciones facilitan la obtención de los multiplicadores finales aplicados en otros sistemas (como `GuildSystem.py`):

- `get_effective_mission_slots(guild)`: Retorna el número de slots de misiones autónomas.
- `get_war_bonus(guild)`: Retorna el bonus porcentual acumulado para resolución de conflictos.
- `get_resource_bonus(guild)`: Retorna el multiplicador total de producción.
- `get_food_efficiency(guild)`: Retorna el multiplicador de duración de suministros.

---

## Interfaz de Usuario

### `open_guild_skill_tree(guild, console)`
Abre una interfaz interactiva de pantalla completa (usando `rich.Live`) que permite navegar por las ramas mediante pestañas.
- **Controles**: Flechas para navegar, Enter para comprar, Esc para salir.
- **Visualización**: Muestra el estado de cada habilidad (✓ Activa, ▶ Seleccionada, Bloqueada) y una descripción detallada del efecto.

---

## Ejemplos de Uso

### Consultar un bonus desde otro módulo

```python
from Modules.ModulesFunctions.GuildSkillTree import get_resource_bonus

# En GuildSystem.py durante la producción
bonus = get_resource_bonus(guild) 
# Si tiene 'eco_logistics' y 'eco_hegemony', bonus será 1.7 (1.0 + 0.2 + 0.5)
```

### Abrir el menú del árbol

```python
from Modules.ModulesFunctions.GuildSkillTree import open_guild_skill_tree

# Llamado desde el menú de la Guild del jugador
open_guild_skill_tree(player_guild, console)
```
