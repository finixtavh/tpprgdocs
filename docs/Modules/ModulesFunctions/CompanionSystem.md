# `CompanionSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `cCompanion`](#clase-ccompanion)
3. [Sistema de Reclutamiento](#sistema-de-reclutamiento)
    - [Costos y Rarezas](#costos-y-rarezas)
    - [Generación de Identidad](#generación-de-identidad)
4. [Gestión de Salarios y Deuda](#gestión-de-salarios-y-deuda)
5. [Combate y Equipamiento](#combate-y-equipamiento)
6. [Resurrección](#resurrección)

---

## Descripción General

`CompanionSystem.py` gestiona a los aliados que el jugador puede contratar para que le asistan en combate. Los compañeros tienen sus propias estadísticas, rarezas, clases y un sistema de mantenimiento económico basado en salarios temporales.

---

## Clase `cCompanion`

Es la clase base para todos los aliados. A diferencia del jugador, tienen un sistema de equipamiento simplificado (un arma y una armadura) y no tienen un árbol de habilidades propio, basando su poder en su nivel y equipo.

### Atributos Clave
- **companion_id**: ID único del catálogo.
- **generated_name**: Nombre aleatorio único asignado al contratar.
- **is_hired**: Indica si el compañero requiere pago de salario.
- **DEF / DefensePercentage**: Arrays de defensa elemental (5 tipos).
- **Barrier**: Escudo protector similar al del jugador.

---

## Sistema de Reclutamiento

### Costos y Rarezas
El costo de contratación y el salario dependen de la rareza del compañero:

| Rareza | Costo Inicial | Salario (cada 3 días) |
|---|---|---|
| **Common** | 50g | 30g |
| **Uncommon** | 120g | 70g |
| **Rare** | 280g | 150g |
| **Epic** | 600g | 320g |
| **Legendary** | 1200g | 700g |

### Generación de Identidad
Al contratar un compañero, se le asigna un género y un nombre aleatorio (Nombre + Apellido) extraído de `Names.json`. Esto dota a cada aliado de una "identidad" única dentro del equipo del jugador.

---

## Gestión de Salarios y Deuda

El sistema cobra automáticamente los salarios basándose en el tiempo real transcurrido (usando `datetime`).
- **Intervalo**: 3 días reales.
- **Límite de Deuda**: Si el oro del jugador es inferior a -100g tras cobrar los salarios, todos los compañeros contratados abandonarán el equipo inmediatamente.

---

## Combate y Equipamiento

### Combate
Los compañeros atacan automáticamente en el turno del jugador.
- Pueden evadir ataques enemigos.
- Pueden realizar contraataques si su estadística `CounterAttack` lo permite.
- Sus daños y defensas se ven afectados por el clima.

### Equipamiento
Se les puede equipar cualquier arma o armadura del inventario del jugador. Al hacerlo, se recalculan sus estadísticas de vida máxima, mana y defensas elementales de forma instantánea.

---

## Resurrección

Si un compañero cae en combate (KO), puede ser revivido de dos formas:
1. **Oro**: Pagando un costo fijo (por defecto 100g) en el menú de gestión.
2. **Objetos**: Usando ítems consumibles marcados con `is_revival_item: true` (recuperan el 100% de HP).

---

## Notas Técnicas

- Utiliza un sistema de caché para el catálogo de compañeros para evitar lecturas constantes de disco.
- Los salarios se calculan de forma retroactiva (si el jugador no abre el juego en 9 días, se le cobrarán 3 ciclos de golpe).
- La persistencia del tiempo se guarda en `Data/Companions/Time.json`.
