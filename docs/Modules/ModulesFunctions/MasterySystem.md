# `MasterySystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `MasterySkill`](#clase-masteryskill)
3. [Gestor: `MasteryManager`](#gestor-masterymanager)
4. [Mecánica de Progresión](#mecánica-de-progresión)
5. [Bonus por Nivel](#bonus-por-nivel)
6. [Interfaz de Usuario](#interfaz-de-usuario)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`MasterySystem.py` implementa el sistema de "Maestrías" de combate. A diferencia de los niveles de personaje, las maestrías suben de forma orgánica al usar equipos específicos. Cuanto más use el jugador una categoría de arma o armadura, más eficiente se volverá con ella.

---

## Clase `MasterySkill`

Representa el nivel de maestría en una categoría específica (ej. "Espadas").
- `current_level`: Nivel actual (1-20).
- `exp_current`: Progreso hacia el siguiente nivel.
- `bonus_multiplier`: Calcula el modificador de daño o defensa basado en el nivel.

---

## Gestor: `MasteryManager`

Coordina todas las maestrías del jugador.

### Categorías de Armas
- Swords (Espadas)
- Staffs (Bastones)
- Hammers (Martillos)
- Spears (Lanzas)
- Daggers (Dagas)
- Bows (Arcos)
- Generic (Armas no categorizadas)

### Categorías de Armaduras
- Helmet (Cascos)
- Armor (Pechos/Armaduras)
- Boots (Botas)
- Ring (Anillos)

---

## Mecánica de Progresión

La experiencia se gana automáticamente durante el combate:
- **Maestría de Armas**: Gana EXP al **infligir daño**.
    - `EXP = Daño_Realizado / 35`.
- **Maestría de Armaduras**: Gana EXP al **recibir daño**.
    - `EXP = Daño_Recibido / 35`.

---

## Bonus por Nivel

Cada nivel de maestría (del 2 al 20) otorga beneficios acumulativos:

| Tipo | Bonus por Nivel | Total al Nv. 20 |
|---|---|---|
| **Armas** | +0.5% Daño | +9.5% Daño |
| **Armaduras** | +0.25% Reducción | +4.75% Reducción |

---

## Interfaz de Usuario

### `display_masteries(console)`
Genera una tabla detallada con `Rich` que muestra:
- El tipo de maestría.
- El nivel actual y la barra de progreso visual `██░░`.
- El bonus exacto de atributo que se está aplicando actualmente.

---

## Ejemplos de Uso

### Registrar daño en el bucle de combate

```python
from Modules.ModulesFunctions.MasterySystem import ensure_mastery

mgr = ensure_mastery(player)

# Al final del turno del jugador
msg = mgr.register_damage_dealt(player, daño_final)
if msg:
    print(msg) # "¡Mastery de Espadas subió a Nv.5!"
```

### Aplicar bonus en el cálculo de daño

```python
mult = mgr.get_weapon_damage_multiplier(player)
daño_total = daño_base * mult
```
---

## Notas Técnicas

- El sistema utiliza un Singleton por jugador almacenado en `player.mastery_manager`.
- El crecimiento de EXP es exponencial: `100 * (1.5 ^ (nv-1))`.
- Las maestrías se guardan y cargan automáticamente a través del `SaveSystem`.
