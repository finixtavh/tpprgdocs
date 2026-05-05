# `input_utils.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Gestión de Listeners (`keyboard_listener_scope`)](#gestión-de-listeners-keyboard_listener_scope)
3. [Menús Interactivos](#menús-interactivos)
    - [`interactive_menu_select()`](#interactive_menu_select)
    - [`select_option_with_arrows()`](#select_option_with_arrows)
4. [Utilidades Simples](#utilidades-simples)
    - [`wait_for_enter()`](#wait_for_enter)
5. [Temas Visuales](#temas-visuales)

---

## Descripción General

`input_utils.py` gestiona toda la interacción por teclado en el juego. Utiliza la librería `sshkeyboard` para capturar pulsaciones de teclas sin necesidad de presionar Enter, y la integra con `rich` para crear menús visualmente atractivos y dinámicos.

---

## Gestión de Listeners (`keyboard_listener_scope`)

`sshkeyboard` tiene una limitación crítica: solo permite un listener activo a la vez. Para evitar errores de tipo `AssertionError`, este módulo implementa un **Context Manager** y un **Lock** global:

```python
with keyboard_listener_scope():
    listen_keyboard(...)
```

Este scope se encarga de detener cualquier listener previo, esperar el tiempo de gracia necesario y serializar las peticiones de entrada de diferentes hilos.

---

## Menús Interactivos

### `interactive_menu_select()`
Crea un menú visual con bordes, títulos y selección resaltada.
- **Navegación**: ↑/↓ para mover el cursor, Enter para seleccionar.
- **Acceso Rápido**: Permite escribir el número de la opción para saltar directamente.
- **Temas**: Soporta temas como `fantasy` (con iconos especiales) o `default` (estilo terminal limpia).
- **Transient**: El menú desaparece de la terminal una vez se ha tomado la decisión.

### `select_option_with_arrows()`
Una versión simplificada para la consola estándar que no requiere el sistema de `Live` de Rich. Muestra el prompt en una sola línea y permite navegar cíclicamente por las opciones.

---

## Utilidades Simples

### `wait_for_enter()`
Pausa la ejecución del hilo actual hasta que el jugador presiona la tecla Enter. Es esencial para dar tiempo al jugador a leer mensajes de log o resultados de combate.

---

## Temas Visuales

El módulo define un `Theme` de Rich global para mantener la coherencia estética:
- `menu_title`: Amarillo negrita.
- `menu_option`: Blanco brillante.
- `success`: Verde negrita.
- `danger`: Rojo negrita.

---

## Notas Técnicas

- La entrada se procesa en hilos (threads) separados para permitir que la UI (`Live`) se refresque independientemente a 30 FPS.
- Incluye un `delay_second_char` de 0.0 para una respuesta instantánea del teclado.
- El sistema de buffering de números permite seleccionar opciones de 2 o más dígitos (ej. "15") de forma natural.
