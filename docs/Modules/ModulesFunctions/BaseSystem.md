# `BaseSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Gestión de Edificios (`Building`)](#gestión-de-edificios-building)
3. [Tipos de Edificios y Efectos](#tipos-de-edificios-y-efectos)
4. [Sistema de Construcción y Materiales](#sistema-de-construcción-y-materiales)
5. [Minijuegos de Entrenamiento](#minijuegos-de-entrenamiento)
    - [Timing (Campo de Entrenamiento)](#timing-campo-de-entrenamiento)
    - [Endurance (Arena de Resistencia)](#endurance-arena-de-resistencia)
6. [Integración con Mastery System](#integración-con-mastery-system)

---

## Descripción General

`BaseSystem.py` permite al jugador construir y mejorar infraestructuras en su base personal. Estos edificios otorgan bonificaciones pasivas globales o permiten acceder a minijuegos que entrenan las maestrías del jugador.

---

## Gestión de Edificios (`Building`)

Cada edificio es una instancia de la clase `Building` que rastrea su nivel actual (0 = no construido) y sus metadatos definidos en `DataBuildings.json`.

### Propiedades Clave
- **is_built**: True si el nivel es > 0.
- **is_max**: True si ha alcanzado el nivel máximo definido.
- **passive_total**: El valor del efecto acumulado según el nivel.

---

## Tipos de Edificios y Efectos

| Edificio | Efecto Pasivo | Beneficio por Nivel |
|---|---|---|
| **Almacén** | Espacio de Inventario | +5 slots |
| **Salón de Exp.** | Multiplicador de EXP | +2.5% EXP Global |
| **Tesorería** | Botín de Oro | +3% Oro en combates |
| **Campo de Entrenamiento** | Minijuego (Armas) | Desbloquea entrenamiento de armas |
| **Arena de Armaduras** | Minijuego (Armaduras) | Desbloquea entrenamiento de defensa |

---

## Sistema de Construcción y Materiales

El sistema de requisitos es flexible y soporta dos modos de validación:
1. **Por Tipo (Modo OR)**: Si el requisito es un tipo conocido (ej. "Wood"), el sistema sumará cualquier ítem que pertenezca a esa categoría (Roble, Pino, etc.).
2. **Por Nombre Exacto**: Si el requisito no es un tipo general, buscará el ítem específico por su nombre.

La función `build_or_upgrade()` deduce automáticamente los materiales del inventario y el oro del jugador al realizar la mejora.

---

## Minijuegos de Entrenamiento

### Timing (Campo de Entrenamiento)
El jugador debe presionar **Enter** cuando un cursor en movimiento (`●`) se encuentre dentro de la zona marcada con estrellas (`★`). El éxito otorga EXP de maestría de armas.

### Endurance (Arena de Resistencia)
El jugador debe "sobrevivir" a 3 oleadas de daño simulado. La reducción de daño se calcula basándose en la defensa real del jugador. El éxito otorga EXP de maestría de armaduras.

---

## Integración con Mastery System

Los edificios de entrenamiento están vinculados directamente al `MasterySystem`. Al ganar un minijuego, el `BaseManager` identifica todos los tipos de armas o armaduras registrados y añade experiencia a sus respectivos niveles de maestría.

---

## Notas Técnicas

- Los edificios se guardan dentro del objeto `player` bajo la clave `player_base`.
- La inyección de edificios desde mods es posible a través de la API del `ModLoader`.
- Los minijuegos utilizan hilos secundarios (`threading`) para manejar animaciones y entrada de teclado simultáneamente.
