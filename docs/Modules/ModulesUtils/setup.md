# `setup.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clase `cPlayer`](#clase-cplayer)
5. [Clase `cEnemy`](#clase-cenemy)
6. [Sistemas de Objetos (`cWeapon`, `cEquippableItems`, `cObject`)](#sistemas-objetos)
7. [Sistema de Inventario (`cInventory`)](#sistema-inventario)
8. [Funciones de Normalización](#funciones-normalizacion)

---

## 1. Descripción General
`setup.py` es el archivo fundacional o *Engine Core Data Structure* de TPPRPG. Define de manera orientada a objetos todas las entidades con estado del juego: el personaje del jugador, los enemigos, los objetos (equipables o consumibles) y el inventario. Expone la API de bajo nivel para las matemáticas de combate y mitigación de daño, además de proporcionar los normalizadores estructurales.

---

## 2. Dependencias e Inyecciones
- **`Modules.ModulesUtils.Loader`**: Utilizado indirectamente, aunque su uso nativo ha sido refactorizado.
- **Inyecciones Condicionales (Lazy/Try-Except)**: Depende dinámicamente de `WeatherSystem` y `GuildSystem` (para procesar multiplicadores globales de combate y `Tax Debuffs`), así como de `EnchantmentSystem` (para interceptar daños en tiempo de cálculo sin generar referencias circulares).

---

## 3. Constantes y Variables Globales
- `player = None`: Singleton temporal del jugador activo durante el Runtime.
- `vInventory = cInventory()`: Instancia estática del inventario del jugador.

---

## 4. Clase `cPlayer`

Clase maestra que gestiona el estado y la progresión del avatar del usuario.

### Estadísticas Principales
- **Health / Health_max**: Vida. Regenera de a poco usando `Check()`.
- **Mana / Mana_max / ManaRegen**: Puntos de habilidad y su velocidad de regeneración por turno.
- **EXP / EXP_M / Level**: Curva de progresión del personaje. Sube estadísticas al invocar `level_up()`.
- **DEF / DefensePercentage**: Arrays de 5 dimensiones `[Physical, Thermal, Earth, Electric, Deep]`.
- **Barrier**: Escudo temporal y ablativo.

### Funciones de Ciclo de Vida
- `Check(inventory)`: La función latido ("Heartbeat"). Restaura Mana, controla el contador de buffs temporales o venenos (`applieddebuffs`), reescala las estadísticas del jugador según su equipamiento y ejecuta la lógica perezosa de los anillos inyectados en la base de datos de Modifiers.
- `level_up()`: Función de subida de nivel. Restablece el HP y aumenta las barreras.

### Lógica de Combate
- `attack(enemy) -> dict`: Inicia el flujo de daño. Genera los daños aleatorios en el arma y aplica modificadores de clima (`WeatherSystem`) e impuestos (`GuildSystem`). Mide probabilidad de "Contraataque" del rival y devuelve un mega-diccionario "Details".
- `TakeDamage(damage_array, attacker=None) -> dict`: El escudo receptor. Escala el ataque recibido y deduce Vida considerando el siguiente orden matemático de prioridad:
  1. Multiplicadores Ambientales de Clima.
  2. Absorción de la `Barrier` activa.
  3. Descuentos porcentuales (`DefensePercentage`).
  4. Descuentos planos (`DEF`).

---

## 5. Clase `cEnemy`

Define el comportamiento y estadísticas de la Inteligencia Artificial de las criaturas. Comparte el molde matemático con el `cPlayer` (tiene `TakeDamage` y `attack`) pero es agnóstico del equipamiento y usa valores codificados en JSON (`DataStats.json`).
- Admite flags como `IS_BOSS` o `SPAWN_CHANCE`.
- Implementa una función de Loot Table que es evaluada en el propio archivo.

---

## 6. Sistemas de Objetos

El motor diferencia rígidamente 3 tipos de interacciones base, y todos se inicializan con la función `create_item_object` del `ItemManager`.

### `cWeapon`
- Contiene los arrays `DMG_min` y `DMG`.
- Su método `use_weapon(player)` arroja N dados de 5 caras virtuales, suma bufos de equipo e inyecta la reducción matemática.

### `cEquippableItems`
- Entidades que aportan bufos pasivos (`HealthBoost`, `Defense`, `DmgBoost`).
- El método `equip(player)` los vincula al Singleton `player`, y delega a `player.Check()` la hidratación de estado global.

### `cObject`
- Objetos consumibles como pociones o materiales puros que no se equipan.
- Atributos como `HealthRestore` y `ManaRestore`.

---

## 7. Sistema de Inventario (`cInventory`)

Sistema de Slots apilables en memoria.
- **`InventorySlot`**: Estructura de datos interna de tupla `(item, quantity)`.
- `addObject(item, quantity)`: Compara el objeto usando `type(item).__name__` y su identificador estático único (`STATIC_ID`). Si hay coincidencia perfecta, incrementa el entero del stack. Si no, añade un nuevo Slot.
- `remove_item(item, quantity)`: Modifica cantidades, previene números negativos y destruye el Slot de la memoria si llega a cero.

---

## 8. Funciones de Normalización

El motor v0.9 usa un vector de 5 elementos para representar el daño/defensa: `[Físico, Térmico, Tierra, Eléctrico, Profundo]`.
Los métodos ocultos de hidratación son:
- `_normalize_5_values(values)`: Inyecta ceros de relleno (`Padding`) a cualquier lista incompleta (ej: `[10] -> [10, 0, 0, 0, 0]`).
- `_normalize_4_values(values)`: Lo mismo pero para daños mínimos, debido a que el atributo número 5 ("Profundo") ignora el RNG y siempre hace daño máximo, descartando el slot mínimo.
