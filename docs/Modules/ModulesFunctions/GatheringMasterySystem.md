# `GatheringMasterySystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)
6. [Flujo del Minijuego de Recolección](#flujo-del-minijuego)

---

## 1. Descripción General
`GatheringMasterySystem.py` define y gestiona el sistema interactivo de recolección de recursos (Minijuegos) y la progresión de las "Gathering Skills" (ej: Minería, Pesca, Tala). Cuando un jugador interactúa con un recurso en el mundo, este módulo orquesta un minijuego en tiempo real (barra de sincronización) donde el jugador puede obtener bonificaciones de progreso al presionar una tecla en el momento exacto. Paralelamente, otorga experiencia pasiva subiendo de nivel dichas profesiones.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere el `config.json` para definir parámetros de balance del minijuego (ej: anchura de la zona segura, velocidad del cursor, intervalos de tick).
- **Inyecciones**:
  - `ModulesUtils.Loader` (para carga de configs).
  - Librerías TUI (`rich.live`, `rich.console`, `sshkeyboard`) para dibujar el minijuego de forma asíncrona.
  - `Modules.ModulesUtils.input_utils` (para atrapar eventos de teclado de forma segura y sin interrupciones).

---

## 3. Constantes y Variables Globales
- Instancia Global Automática: `Gathering_init` es una variable instanciada a nivel módulo que crea la clase `cFarm`. Funciona como el Singleton de facto.

---

## 4. Clases y Estructuras de Datos

### `cSkill`
Representa la progresión de una profesión específica.

#### Constantes Base de Nivelación
- `MAX_LEVEL` (`int`): Nivel máximo (50).
- `EXP_BASE` (`float`): Experiencia inicial para nivel 2 (100.0).
- `EXP_GROWTH` (`float`): Multiplicador de curva de experiencia (1.5).
- `BONUS_EFFICIENCY` (`float`): +5% de poder base de recolección extra por cada nivel ganado.
- `BONUS_XP_FACTOR` (`float`): +10% de ganancia de EXP multiplicativa por cada nivel.

#### Atributos de Instancia
- `self.exp_total` (`float`): Experiencia acumulada actual.
- `self.exp_to_next_lvl` (`float`): Experiencia requerida calculada de la curva.
- `self.current_level` (`int`): Nivel de profesión.

#### Métodos Principales
- `add_exp(self, amount: float) -> int`: Sube EXP y hace Level Up si supera el límite. Retorna la cantidad de niveles ganados de golpe (en caso de otorgar mucha EXP).
- **Propiedades Computadas**: `efficiency_bonus`, `xp_bonus_factor` devuelven el multiplicador pasivo que se utilizará en las matemáticas del minijuego.

### `cFarm`
Motor central de habilidades, inicializado como `Gathering_init`.

#### Constantes de Minijuego (Cargadas de `config.json`)
- `ZONE_BONUS` (`float`): Multiplicador al presionar la tecla en la zona verde (ej: 0.2).
- `TICK_INTERVAL` (`float`): Velocidad de renderizado del bucle Live.
- `MARKER_SPEED` / `ZONE_WIDTH_MIN` / `ZONE_WIDTH_MAX`: Variables procedimentales para la dificultad del minijuego.

#### Atributos de Instancia
- `self.gathering` (`Dict[str, cSkill]`): Diccionario con las profesiones activas (`mining`, `woodcutting`, `fishing`, `herbalism`, `skinning`, `astrology`).

#### Métodos Principales
- `_build_display(...) -> Panel`: Renderizador interno de la UI del minijuego. Compone las barras ASCII y formatea la tabla central con *rich*.
- `gather(self, material, tool, buffs) -> int`: Método maestro. Orquesta el multihilo, bloquea la entrada del jugador, evalúa las pulsaciones, restaura o destruye herramientas (basado en durabilidad), y calcula los yields. Retorna el número del botín final recolectado.

---

## 5. Funciones del Módulo (API)

- El módulo opera principalmente instanciando `cMaterial` y `cTool` importados desde este archivo por conveniencia por otros scripts, y luego disparando `Gathering_init.gather()`. 
- *(Nota: Las definiciones reales de `cMaterial` y `cTool` fueron trasladadas a `ItemManager` por arquitectura, pero este script contiene la lógica de consumo).*

---

## 6. Flujo del Minijuego de Recolección

1. **Iniciación**: El jugador interactúa con una fuente de recursos. El evento emite la función `gather` pasándole el material base y la herramienta equipada (ej: `Iron Pickaxe`).
2. **Validación**: Comprueba si el jugador tiene el nivel mínimo de la profesión en `cSkill`.
3. **Cálculo de Poder**: Multiplica la `Tool.Efficiency` $\times$ `Skill.EfficiencyBonus` $\times$ `Buffs Externos`. Esto define la cantidad total de Tiempo que el jugador tendrá que mantener la pantalla abierta.
4. **Bucle Asíncrono (`rich.Live`)**: Se abre un `threading` con la función `listen_keyboard`.
5. **Mecánica**: 
   - El tiempo baja pasivamente.
   - Un marcador `▼` se desliza sobre una barra.
   - Si el jugador presiona `ESPACIO` en la zona `VERDE`, el sistema le otorga `ZONE_BONUS` de progreso extra y le resta tiempo faltante, acortando la recolección.
6. **Finalización**: Se detiene el thread de teclado, se aplica daño a la durabilidad de la `cTool` (destruyéndola si llega a 0), y se suma el valor de botín multiplicando `Yield` de herramienta y posibles drops críticos. El material cae directo al inventario externo.
