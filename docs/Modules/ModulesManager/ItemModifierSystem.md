# `ItemModifierSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Sistemas de Roll y Probabilidad](#sistemas-de-roll-y-probabilidad)
3. [Tipos de Modificadores](#tipos-de-modificadores)
4. [Integración y Aplicación](#integración-y-aplicación)

---

## Descripción General

El `ItemModifierSystem` (IMS) es el motor encargado de generar y aplicar afijos y sufijos aleatorios (modifiers) a las armas, equipables y herramientas que se obtienen en el juego (ya sea por crafteo, drops de combates, o tesoros). Su objetivo es añadir variedad procedural a la itemización del juego.

La lectura base de probabilidades o calidad se ve influenciada por los modificadores globales de configuración definidos en `config.json` (`GLOBAL_CRAFTING_QUALITY_MODIFIER`, `GLOBAL_LOOT_MODIFIER`, etc.).

---

## Sistemas de Roll y Probabilidad

El sistema decide la "rareza" de un modificador aplicando un sistema de "tiradas" (roll). 
Actualmente, las categorías de modificadores están divididas en:

1. **Malos (Bad)**: Afectan negativamente al arma (ej. "Roto", "Maldito"). Se dan si la suerte es extremadamente baja o como mecánica de riesgo.
2. **Normales (Normal/Standard)**: Mejoras moderadas (+20% de daño, +10 Defensa, etc.).
3. **Buenos (Good/Epic/Legendary)**: Mejoras drásticas, con probabilidades bajas (ej. "Divino", "Vorpal", "Indestructible").

La función principal `roll_and_apply_modifier(item, roll_bonus=0.0)` realiza lo siguiente:
- Extrae un número aleatorio entre 0.0 y 1.0 y le suma el `roll_bonus`.
- Si el resultado final es < 0.20, aplica un modificador malo.
- Si es >= 0.20 y < 0.85, aplica uno normal.
- Si es >= 0.85, aplica uno bueno.

---

## Tipos de Modificadores

Los modificadores alteran permanentemente los atributos del `item` cuando se aplican y modifican el nombre (`item.name = f"{mod_name} {item.name}"`). 

A continuación un resumen de las estadísticas que pueden alterar:
- **Armas (`cWeapon`)**: Modifica la matriz de `DMG` multiplicando los valores base (ej. daño físico x1.5, daño eléctrico x2). Cambia el `Gold_Cost`.
- **Equipables (`cEquippableItems`)**: Multiplica los valores del array `DEF` (defensas) y altera el oro.
- **Herramientas (`cTool`)**: Incrementa o reduce la `Speed` (velocidad de minijuego de recolección), `Yield` (recolección extra) o `Durability`.

---

## Integración y Aplicación

El sistema está profundamente integrado en varias fases del flujo de juego:

1. **Crafteo**: Cuando el jugador usa el yunque (`CraftingSystem.py`), los objetos crafteados reciben una tirada automática de modificador.
2. **Reforjado (Anvil Reroll)**: La mecánica de Reforjado en el yunque permite re-rollear el modificador de un objeto existente pagando oro. El anterior modificador se borra (revirtiendo el nombre temporalmente) y se aplica el nuevo roll.
3. **Loot Drops**: Enemigos y tesoros pasan los objetos generados a través de `roll_and_apply_modifier` con bonificaciones de suerte dependiendo de la dificultad de la zona.
