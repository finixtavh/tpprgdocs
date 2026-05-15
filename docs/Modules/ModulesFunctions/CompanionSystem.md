# `CompanionSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Mecánicas de Economía y Pagos](#mecánicas-de-economía-y-pagos)

---

## 1. Descripción General
El `CompanionSystem.py` maneja la instanciación, combate, equipamiento y economía de los "Compañeros" (NPCs aliados). Los compañeros pueden pelear junto al jugador, poseen sus propias estadísticas (HP, Mana, Defensas, etc.) y consumen oro periódicamente como salario si fueron "contratados". 

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: 
  - `./Data/DataCompanions.json`: Define las plantillas de NPCs (Rareza, Costos base, Armas por defecto).
  - `./Data/Names.json`: Diccionarios de nombres aleatorios segmentados por género y apellidos.
  - `./Data/Companions/Time.json`: Archivo persistente separado para llevar el control del último cobro de salarios.
- **Inyecciones Condicionales**: 
  - Funciones desde `Modules.ModulesUtils.setup` (ej: `apply_weather_combat_mods`) inyectadas en tiempo de ejecución para modificar el Daño/Evasión según el sistema de clima.

---

## 3. Constantes y Variables Globales
- `COMPANION_REVIVAL_GOLD_COST` (`int`): Oro necesario para resucitar a un aliado (100).
- `COMPANION_PAY_INTERVAL_DAYS` (`int`): Ciclos de cobro (cada 3 días de tiempo real).
- `COMPANION_DEBT_LIMIT` (`float`): Tope de deuda ($-100.0$). Si la deuda baja de este número, los compañeros desertan.
- `RARITY_HIRE_COST` (`Dict[str, int]`): Oro para contratación inicial, clasificado por rareza (`Common` a `Legendary`).
- `RARITY_PAY_COST` (`Dict[str, int]`): Oro del cobro recurrente cada 3 días, por rareza.

---

## 4. Clases y Estructuras de Datos

### `_DefaultWeapon` / `_DefaultArmor`
Clases wrapper internas. Permiten que los NPCs cargados desde el JSON tengan un equipo "simulado" sin necesidad de instanciar objetos completos desde el `ItemManager`. Solo guardan daño/defensas brutas y bonificadores de estadísticas.

### `cCompanion`
Entidad principal del aliado. Actúa de forma análoga a la clase del Jugador en el ciclo de combate.

#### Atributos de Instancia (Relevantes)
- `self.companion_id` (`str`): ID de plantilla base.
- `self.name` (`str`): Tipo de clase u oficio (Ej: "Arquero").
- `self.generated_name` (`str`): Nombre procedimental (Ej: "Kael Stormcrest").
- `self.is_hired` (`bool`): Bandera para indicar si cobra salario.
- `self.Health` / `self.Health_max` / `self.Mana` / `self.Mana_max` (`float`): Estadísticas vitales.
- `self.DEF` (`List[float]`): Array de 5 elementos para defensas Físico, Mágico, Elemental, Perforante, Divino.
- `self.weapon` / `self.armor` (`Optional[object]`): Referencias al equipo real o simulado.

#### Métodos de Equipamiento
- `equip_weapon(self, weapon)` / `equip_armor(self, armor)`
  - **Propósito**: Muta el inventario equipado. Quita los bonos pasivos del ítem anterior y aplica los del nuevo recalculando los máximos de HP/Mana.

#### Métodos de Combate
- `attack(self, enemy) -> dict`
  - **Propósito**: Realiza la fase ofensiva. Calcula Evasión y Contraataque del enemigo (afectado por clima). Suma daños mínimos/máximos del arma o lanza el cálculo avanzado de la misma (`weapon.use_weapon()`).
  - **Retornos**: Diccionario detallado con daño provocado, mitigación, absorción de barreras, etc.
- `TakeDamage(self, damage, attacker=None)`
  - **Propósito**: Fase defensiva. Absorbe daño mediante `self.Barrier`, aplica resistencias absolutas (`self.DEF/2`) y luego bloqueos por porcentaje (`DefensePercentage`).
  - **Mutación**: Resta HP y cambia `self.is_alive` a `False` si llega a 0.

---

## 5. Funciones del Módulo (API)

### Gestión y Carga
- `get_companion_data() -> dict`: Parsea el `DataCompanions.json`.
- `load_companion_by_id(companion_id: str) -> Optional[cCompanion]`: Constructor universal. Instancia la clase `cCompanion` utilizando las stats de la plantilla.
- `assign_random_identity(companion)`: Extrae nombres de `Names.json` dependiendo de una tirada de género y se los asigna al NPC.

### Economía
- `process_companion_payments(player, console)`: Revisa el `Time.json`. Si pasaron ciclos vencidos de 3 días, descuenta oro al jugador por cada aliado contratado. Desertan si no puede pagar.
- `hire_companion(companion, player, console) -> bool`: Ejecuta la transferencia de oro inicial de contratación. Devuelve booleano.

### Menús Interactivos
- `manage_companions_menu(player, inventory, console, main_menu_fn, pause_fn)`: Panel central que permite curar, resucitar, cambiar equipo o despedir a los compañeros activos.
- `hire_companion_menu(...)`: Muestra las "Tabernas" categorizadas por rareza para que el jugador gaste su oro en nuevos NPC aleatorios de `DataCompanions.json`.

---

## 6. Mecánicas de Economía y Pagos

El sistema económico está desacoplado del "tiempo ingame" y atado al **Tiempo Real** a través de `Time.json`.
1. **Contratación**: El jugador paga un monto de enganche alto (`RARITY_HIRE_COST`).
2. **Ciclo de Cobro (`calculate_payments_due`)**: Se calcula matemáticamente cuántos periodos de 3 días han pasado desde la última revisión (sin importar si el juego estuvo cerrado).
3. **Cobro Múltiple**: Si el jugador dejó el juego cerrado 10 días, el sistema le cobrará *3 ciclos* completos de golpe.
4. **Deserción**: Si al cobrar, el jugador baja de `-100 de Oro`, sus aliados contratados desaparecen del array `player.companion`.
