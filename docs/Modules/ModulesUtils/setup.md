# `setup.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `cPlayer`](#clase-cplayer)
3. [Clase `cEnemy`](#clase-cenemy)
4. [Sistemas de Objetos](#sistemas-de-objetos)
    - [`cWeapon`](#cweapon)
    - [`cEquippableItems`](#cequippableitems)
    - [`cObject`](#cobject)
5. [Sistema de Inventario (`cInventory`)](#sistema-de-inventario-cinventory)
6. [Funciones de Normalización](#funciones-de-normalización)

---

## Descripción General

`setup.py` es el núcleo de datos del juego. Define las clases fundamentales para el jugador, los enemigos y los objetos, así como la lógica base de combate (ataque y recepción de daño) y el sistema de inventario con stacking.

---

## Clase `cPlayer`

Representa al personaje del jugador. Gestiona estadísticas, equipamiento y progresión.

### Estadísticas Principales
- **Health / Health_max**: Puntos de vida.
- **Mana / Mana_max**: Puntos de energía para habilidades.
- **EXP / EXP_M**: Experiencia actual y requerida para el siguiente nivel.
- **DEF**: Array de 5 valores de defensa plana.
- **DefensePercentage**: Array de 5 valores de reducción porcentual.
- **Barrier**: Escudo temporal que absorbe daño antes que la salud.

### Métodos Clave
- `attack(enemy)`: Calcula el daño saliente aplicando clima y encantamientos.
- `TakeDamage(damage, attacker)`: Procesa el daño entrante siguiendo el orden: Barrera → Encantamientos → Defensa Plana → Defensa Porcentual.
- `level_up()`: Incrementa estadísticas y otorga puntos de habilidad al alcanzar `EXP_M`.
- `Check(inventory)`: Actualiza estados, regenera mana y sincroniza el equipamiento.

---

## Clase `cEnemy`

Define el comportamiento y estadísticas de los monstruos.
- **DMG / DMG_min**: Arrays de daño para los 5 tipos elementales.
- **DEF**: Array de defensa elemental.
- **EXP_TO_PLAYER / GOLD**: Recompensas otorgadas al morir.
- **IS_BOSS**: Flag para enemigos especiales con mecánicas únicas.

---

## Sistemas de Objetos

### `cWeapon`
Armas que definen el daño del jugador.
- `use_weapon(player)`: Genera un diccionario de daño aleatorio entre `DMG_min` y `DMG` para cada tipo elemental.

### `cEquippableItems`
Armaduras, cascos, botas y anillos.
- **Slots**: Casco (Helmet), Armadura (Armor), Botas (Boots), Anillos (Ring 1, 2, 3).
- **AppliedStats**: Asegura que las estadísticas se sumen al jugador solo una vez al equipar.

### `cObject`
Objetos consumibles (pociones, pergaminos).
- **HealthRestore / ManaRestore**: Valores de recuperación inmediata.

---

## Sistema de Inventario (`cInventory`)

Implementa un sistema de slots con **stacking** (apilamiento).
- **InventorySlot**: Contenedor para un tipo de ítem y su cantidad.
- **addObject(item, quantity)**: Busca slots existentes del mismo ítem para apilar o crea nuevos si no hay espacio.
- **remove_item(item, quantity)**: Reduce la cantidad de los slots y los elimina si llegan a cero.

---

## Funciones de Normalización

Para garantizar la compatibilidad con el sistema de 5 tipos de daño/defensa:
- `_normalize_5_values()`: Convierte entradas simples o listas cortas en un array de longitud 5.
- `_normalize_4_values()`: Similar, pero para daños mínimos (el daño "Profundo" no tiene valor mínimo aleatorio, es siempre máximo).

---

## Notas Técnicas

- El sistema de daño utiliza `float` para cálculos precisos de buffs/clima antes de aplicar el daño final a los enteros de salud.
- Las dependencias de `WeatherSystem` y `EnchantmentSystem` se cargan de forma diferida (lazy import) para evitar ciclos de importación.
