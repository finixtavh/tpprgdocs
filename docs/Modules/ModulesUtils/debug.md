# `debug.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Constantes](#constantes)
3. [Funciones Principales](#funciones-principales)
4. [Funciones Auxiliares Internas](#funciones-auxiliares-internas)
5. [Ejemplos de Uso](#ejemplos-de-uso)
6. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`debug.py` es el módulo de **logging y diagnóstico** del juego. Configura el sistema de registro de eventos, conecta la consola `rich` al logger para que los mensajes visuales queden también persistidos en disco, captura excepciones no manejadas y puede volcar un snapshot completo del estado de ejecución en tiempo real.

Todos los archivos de log se guardan en el directorio `logs/` con el formato `tpprpg_YYYYMMDD.log`.

---

## Constantes

| Constante | Tipo | Valor | Descripción |
|---|---|---|---|
| `LOGGER_NAME` | `str` | `"TPPRPG"` | Nombre del logger raíz del proyecto. Se usa en todos los módulos que necesiten obtener el mismo logger. |

---

## Funciones Principales

### `setup_logging(debug_enabled=False) -> logging.Logger`

Configura el sistema de logging del proyecto y retorna el logger raíz.

**Comportamiento:**

1. Crea el directorio `logs/` si no existe.
2. Obtiene (o crea) el logger con el nombre `LOGGER_NAME`.
3. Limpia handlers previos para evitar duplicados si se llama varias veces.
4. Crea un `FileHandler` que escribe en `logs/tpprpg_YYYYMMDD.log` en modo *append*.
5. Si `debug_enabled=True`, el nivel mínimo es `DEBUG`; de lo contrario, `INFO`.
6. El formato del log incluye timestamp, nivel, nombre del logger y mensaje.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `debug_enabled` | `bool` | Si `True`, registra también mensajes de nivel `DEBUG`. |

**Retorno:** Instancia de `logging.Logger` configurada y lista para usarse.

```python
from Modules.debug import setup_logging

logger = setup_logging(debug_enabled=True)
logger.info("Sistema inicializado")
logger.debug("Valor de variable x: 42")
```

---

### `attach_console_logging(console, logger) -> None`

Parchea el método `console.print` de una instancia de `rich.Console` para que **todos los mensajes impresos por pantalla también se registren en el log** según su contenido.

**Detección automática de nivel:**

| Contenido del mensaje | Nivel de log |
|---|---|
| Contiene `[danger]`, `[red]` o `"error"` | `logger.error(...)` |
| Contiene `[warning]` o `"warning"` | `logger.warning(...)` |
| Contiene `[success]` o `[green]` | `logger.info(...)` |
| Contiene `[info]` | `logger.info(...)` |
| Ninguno de los anteriores | No se registra |

Los tags de marcado Rich (`[bold]`, `[/red]`, etc.) se eliminan del texto antes de guardarlo en el log mediante `_strip_rich_markup()`.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `console` | `rich.Console` | Instancia cuyo método `print` será parcheado. |
| `logger` | `logging.Logger` | Logger donde se registrarán los mensajes. |

```python
from rich.console import Console
from Modules.debug import attach_console_logging, setup_logging

logger  = setup_logging()
console = Console()
attach_console_logging(console, logger)

console.print("[success]Item cargado correctamente[/success]")
# → Se imprime en pantalla Y se guarda en el log como INFO
```

---

### `install_exception_hooks(logger) -> None`

Registra un hook global para capturar **excepciones no manejadas** (`sys.excepthook`). Cuando ocurre una excepción que no es atrapada por ningún `try/except`, se registra en el log con nivel `CRITICAL` incluyendo el traceback completo, en lugar de simplemente mostrarse en la terminal y perderse.

Las excepciones `KeyboardInterrupt` (Ctrl+C) pasan al hook original sin registrarse.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `logger` | `logging.Logger` | Logger donde se registrará la excepción crítica. |

```python
from Modules.debug import setup_logging, install_exception_hooks

logger = setup_logging()
install_exception_hooks(logger)
# A partir de aquí, cualquier excepción no capturada queda en el log
```

---

### `dump_debug_snapshot(*, player, inventory, mod_loader, mod_api, current_state, chosen_area) -> str`

Genera y retorna un **string de diagnóstico** con el estado actual del juego. Útil para el menú DEBUG (opción 12 de `MainGame.py`) o para registrar en el log en momentos clave.

**Información incluida:**

- Estado actual del juego (`current_state`) y zona seleccionada.
- Lista de mods cargados y cantidad.
- Lista de comandos personalizados de mods.
- Número de items en el inventario.
- Todos los atributos no-callable del jugador (hasta 40).

**Parámetros (todos keyword-only):**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `player` | `cPlayer` | Objeto del jugador. |
| `inventory` | `cInventory` | Inventario del jugador. |
| `mod_loader` | `ModLoader` | Instancia del ModLoader. |
| `mod_api` | `ModAPI` | Instancia de la ModAPI. |
| `current_state` | `GameState` | Estado actual del enum `GameState`. |
| `chosen_area` | `str \| None` | Clave de la zona actual. |

**Retorno:** `str` con el snapshot formateado listo para imprimir o loggear.

```python
from Modules.debug import dump_debug_snapshot

snapshot = dump_debug_snapshot(
    player=player,
    inventory=vInventory,
    mod_loader=mod_loader,
    mod_api=mod_api,
    current_state=current_state,
    chosen_area=chosen_area
)
print(snapshot)
```

---

### `log_imported_modules(logger) -> None`

Registra en el log (nivel `DEBUG`) la lista completa de módulos Python importados en el proceso actual. Solo se ejecuta si el logger tiene nivel `DEBUG` activo.

Útil para diagnosticar conflictos de importación o verificar que los mods se cargaron en `sys.modules`.

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `logger` | `logging.Logger` | Logger donde se registrarán los módulos. |

---

## Funciones Auxiliares Internas

### `_strip_rich_markup(text) -> str`

*(Función interna)* Elimina todas las etiquetas de marcado de Rich (como `[bold]`, `[/red]`, `[success]`) de un string usando una expresión regular. Retorna el texto plano resultante.

```python
_strip_rich_markup("[bold green]Éxito[/bold green]")
# → "Éxito"
```

### `_safe_attrs(obj, max_items=40) -> Dict[str, Any]`

*(Función interna)* Extrae de forma segura hasta `max_items` atributos no-callable y no privados de un objeto. Captura excepciones individuales sin interrumpir el proceso. Usado internamente por `dump_debug_snapshot`.

---

## Ejemplos de Uso

### Inicialización completa al arrancar el juego

```python
from rich.console import Console
from Modules.debug import setup_logging, attach_console_logging, install_exception_hooks

logger  = setup_logging(debug_enabled=False)
console = Console()

attach_console_logging(console, logger)
install_exception_hooks(logger)

logger.info("Juego iniciado")
```

### Generar snapshot en el menú DEBUG

```python
from Modules.debug import dump_debug_snapshot

snapshot = dump_debug_snapshot(
    player=player,
    inventory=vInventory,
    mod_loader=get_mod_loader(),
    mod_api=get_mod_api(),
    current_state=current_state,
    chosen_area=chosen_area,
)
print(snapshot)
logger.debug(snapshot)
```

### Activar modo debug con log detallado

```python
logger = setup_logging(debug_enabled=True)
log_imported_modules(logger)
# → Registra cada módulo importado en el log
```

---

## Notas y Referencias

- El logger usa `propagate = False` para evitar que los mensajes suban al logger raíz de Python y se dupliquen.
- Los handlers se limpian en cada llamada a `setup_logging()` con `logger.handlers.clear()`, lo que hace seguro llamarla varias veces sin acumular handlers.
- **`logging` de Python:** [https://docs.python.org/3/library/logging.html](https://docs.python.org/3/library/logging.html)
- **`sys.excepthook`:** Hook que Python llama cuando una excepción no es capturada. [https://docs.python.org/3/library/sys.html#sys.excepthook](https://docs.python.org/3/library/sys.html#sys.excepthook)
- **`re.sub` para eliminar tags:** La regex `r"\[/?[^\]]+\]"` captura cualquier tag con formato `[...]` o `[/...]`. [https://docs.python.org/3/library/re.html](https://docs.python.org/3/library/re.html)
