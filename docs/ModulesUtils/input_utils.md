# `input_utils.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Tema Visual Compartido](#tema-visual-compartido)
3. [Gestión del Listener de Teclado](#gestión-del-listener-de-teclado)
4. [Funciones de Selección](#funciones-de-selección)
5. [Funciones de Espera](#funciones-de-espera)
6. [Ejemplos de Uso](#ejemplos-de-uso)
7. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`input_utils.py` centraliza todos los **controles de entrada por teclado** del juego que usan la librería `sshkeyboard`. Proporciona menús interactivos con navegación por flechas, un mecanismo de serialización para evitar conflictos entre listeners, y utilidades de espera de confirmación.

Todos los menús del juego (principal, equipamiento, tienda, etc.) utilizan estas funciones para garantizar una experiencia de usuario consistente.

> **Dependencia principal:** `sshkeyboard` — permite escuchar pulsaciones de tecla sin necesitar permisos de root en Linux (a diferencia de `keyboard`).

---

## Tema Visual Compartido

```python
_MENU_THEME = Theme({
    "info":        "dim cyan",
    "warning":     "magenta",
    "danger":      "bold red",
    "success":     "bold green",
    "menu_title":  "bold yellow",
    "menu_option": "bright_white",
})
```

Este tema es compartido con `MainGame.py` para mantener coherencia visual en todos los menús Rich del juego. Se puede usar al crear una instancia de `Console`:

```python
from rich.console import Console
console = Console(theme=_MENU_THEME)
```

---

## Gestión del Listener de Teclado

### `_MENU_LISTENER_LOCK`

`threading.Lock` global que garantiza que solo **un listener de teclado esté activo a la vez**. `sshkeyboard` lanza un `AssertionError` si se intenta iniciar un segundo listener mientras hay uno activo.

---

### `_stop_keyboard_listener_and_wait() -> None`

*(Función interna)* Llama a `stop_listening()` e introduce una pausa de `0.08` segundos para asegurar que el hilo del listener anterior termine completamente antes de iniciar uno nuevo.

---

### `keyboard_listener_scope()`

**Context manager** que serializa cualquier uso de `listen_keyboard`. Adquiere el lock global, detiene cualquier listener activo antes de entrar, y vuelve a detenerlo al salir.

```python
with keyboard_listener_scope():
    listen_keyboard(on_press=mi_handler, sequential=False)
```

**Uso obligatorio:** Cualquier código que llame a `listen_keyboard` directamente debe hacerlo dentro de este context manager para evitar el error `AssertionError: Only one listener allowed at a time` al encadenar menús.

---

## Funciones de Selección

### `select_option_with_arrows(prompt, valid_options, default_option=None) -> int`

Selector numérico ligero con navegación por flechas. Muestra el prompt en línea (`\r`) y actualiza en tiempo real sin usar `rich.Live`.

**Controles:**

| Tecla | Acción |
|---|---|
| `↑` | Mover selección hacia arriba (cíclico). |
| `↓` | Mover selección hacia abajo (cíclico). |
| `0-9` | Añadir dígito al buffer numérico. |
| `Backspace` | Borrar último dígito del buffer. |
| `Enter` | Confirmar selección (buffer numérico o cursor actual). |

**Parámetros:**

| Parámetro | Tipo | Descripción |
|---|---|---|
| `prompt` | `str` | Texto a mostrar antes del indicador de selección. |
| `valid_options` | `List[int]` | Lista de valores enteros válidos. Se ordenan automáticamente. |
| `default_option` | `int \| None` | Opción seleccionada por defecto al abrir. |

**Retorno:** El entero seleccionado por el usuario.

> **Nota:** Esta función es adecuada para selecciones simples. Para menús con etiquetas y panel visual, usar `interactive_menu_select`.

```python
from Modules.input_utils import select_option_with_arrows

opciones = [1, 2, 3, 4, 0]
elegida = select_option_with_arrows("Elige: ", opciones, default_option=1)
```

---

### `interactive_menu_select(title, options, prompt, subtitle, default_value, allow_zero_shortcut, border_style) -> int`

Menú interactivo completo con panel Rich, navegación por flechas y resaltado de selección con **colores invertidos** (texto negro sobre fondo blanco).

**Controles:**

| Tecla | Acción |
|---|---|
| `↑` | Mover cursor hacia arriba (cíclico). |
| `↓` | Mover cursor hacia abajo (cíclico). |
| `0-9` | Añadir dígito al buffer numérico. |
| `Backspace` | Borrar último dígito del buffer. |
| `Enter` | Confirmar selección (buffer numérico tiene prioridad sobre cursor). |
| `0` + `Enter` | Si `allow_zero_shortcut=True` y `0` es opción válida, selecciona inmediatamente. |

**Parámetros:**

| Parámetro | Tipo | Default | Descripción |
|---|---|---|---|
| `title` | `str` | — | Título del panel del menú. |
| `options` | `List[Tuple[int, str]]` | — | Lista de `(valor, etiqueta)` para cada opción. |
| `prompt` | `str` | `"Selecciona una opción: "` | Texto del pie del menú. |
| `subtitle` | `str \| None` | `None` | Subtítulo opcional debajo del título. |
| `default_value` | `int \| None` | `None` | Valor cuya fila estará seleccionada al abrir. |
| `allow_zero_shortcut` | `bool` | `True` | Si `True`, escribir `0` + Enter selecciona inmediatamente la opción 0. |
| `border_style` | `str` | `"menu_title"` | Color/estilo del borde del panel Rich. |

**Retorno:** El entero (`valor`) de la opción seleccionada.

**Implementación interna:**
- Usa `rich.Live` con 30 fps de refresco para actualizar el panel en tiempo real.
- El listener de teclado corre en un hilo daemon separado.
- Al salir, el panel es `transient=True` (se borra de la terminal).
- Siempre hace `join()` del hilo del listener antes de retornar para evitar condiciones de carrera.

```python
from Modules.input_utils import interactive_menu_select

opciones = [
    (1, "Explorar zona"),
    (2, "Ir a la tienda"),
    (3, "Ver estadísticas"),
    (0, "Salir"),
]
eleccion = interactive_menu_select(
    title="Menú Principal",
    options=opciones,
    default_value=1,
    border_style="yellow",
)
```

---

## Funciones de Espera

### `wait_for_enter() -> None`

Bloquea la ejecución hasta que el usuario presione `Enter`. Imprime `"Presiona Enter"` en la consola y usa `keyboard_listener_scope` internamente.

```python
from Modules.input_utils import wait_for_enter

print("Operación completada.")
wait_for_enter()
# → Continúa al presionar Enter
```

---

## Ejemplos de Uso

### Menú de confirmación (Sí/No)

```python
from Modules.input_utils import interactive_menu_select

confirmar = interactive_menu_select(
    title="Confirmar compra",
    subtitle="¿Deseas comprar este item?",
    options=[(1, "Sí"), (0, "No")],
    prompt="Confirmación: ",
    default_value=1,
    border_style="yellow",
)

if confirmar == 1:
    print("Compra realizada.")
else:
    print("Compra cancelada.")
```

### Selector de zona

```python
from Modules.input_utils import interactive_menu_select

zonas = [
    (1, "Bosque"),
    (2, "Cueva"),
    (3, "Desierto"),
    (0, "Volver"),
]
zona = interactive_menu_select(
    title="Seleccionar Zona",
    options=zonas,
    border_style="green",
)
```

### Uso directo del context manager

```python
from Modules.input_utils import keyboard_listener_scope
from sshkeyboard import listen_keyboard, stop_listening

resultado = {"tecla": None}

def mi_handler(key):
    resultado["tecla"] = key
    stop_listening()

with keyboard_listener_scope():
    listen_keyboard(on_press=mi_handler, sequential=True)

print(f"Tecla presionada: {resultado['tecla']}")
```

---

## Notas y Referencias

- **`sshkeyboard`** es la librería de captura de teclado usada por este módulo. A diferencia de `keyboard`, no requiere permisos de root en Linux. Documentación: [https://github.com/sagi-z/sshkeyboard](https://github.com/sagi-z/sshkeyboard)
- **`threading.Lock`:** Primitiva de sincronización para evitar condiciones de carrera. [https://docs.python.org/3/library/threading.html#threading.Lock](https://docs.python.org/3/library/threading.html#threading.Lock)
- **`rich.Live`:** Componente que actualiza contenido en la terminal en tiempo real. [https://rich.readthedocs.io/en/stable/live.html](https://rich.readthedocs.io/en/stable/live.html)
- **`contextlib.contextmanager`:** Decorador para crear context managers con `yield`. [https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager](https://docs.python.org/3/library/contextlib.html#contextlib.contextmanager)
- El parámetro `sequential=False` en `listen_keyboard` permite procesar múltiples pulsaciones rápidas sin esperas entre ellas, lo que mejora la respuesta del menú.
