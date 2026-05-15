# `input_utils.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Temas](#constantes)
4. [Gestión de Listeners (`keyboard_listener_scope`)](#gestion-listeners)
5. [API de Interfaces y Menús](#api-menus)
6. [Flujos Críticos e Interacción Asíncrona](#flujos-criticos)

---

## 1. Descripción General
`input_utils.py` es el módulo especializado en la captura de eventos de teclado de bajo nivel y la renderización de menús interactivos. Utiliza `sshkeyboard` para permitir inputs "raw" sin que el jugador necesite presionar la tecla `ENTER`, logrando así una navegación TUI (Text User Interface) mucho más fluida. Además, envuelve todas estas interacciones con paneles y tablas estilizadas de la librería `rich`.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Ninguno. Es un módulo de utilidad de nivel inferior del que dependen casi todos los menús visuales (`GuildMenu`, `InventoryMenu`, `MainGame`, etc.).
- **Librerías TUI**:
  - `rich.console`, `rich.panel`, `rich.table`, `rich.text`, `rich.theme`, `rich.live`.
  - `sshkeyboard.listen_keyboard`, `sshkeyboard.stop_listening`.
- **Hilos (`threading`)**: Se usa intensamente para separar la captura de teclas del bucle de renderizado visual (`Live`).

---

## 3. Constantes y Temas
- `_MENU_THEME`: Una constante que inicializa la clase `Theme` de `rich`. Define estilos predeterminados (como `"danger": "bold red"`) utilizados a través del motor.
- `_MENU_LISTENER_LOCK`: Un candado `threading.Lock()` global. Fundamental para prevenir las colisiones que harían crashear el motor si se lanzan múltiples interfaces al mismo tiempo.

---

## 4. Gestión de Listeners (`keyboard_listener_scope`)
La librería `sshkeyboard` tiene una restricción estricta de arquitectura: arroja un `AssertionError` si se intenta instanciar más de un *listener* a la vez.

Para resolver esto, se diseñó el context manager **`@contextmanager keyboard_listener_scope`**:
1. Bloquea el hilo usando el `_MENU_LISTENER_LOCK`.
2. Fuerza la detención asíncrona de cualquier listener huérfano llamando a la subrutina segura `_stop_keyboard_listener_and_wait()`.
3. Inyecta un micro-delay de `0.08` segundos (tiempo necesario para que el sistema operativo purgue el buffer del teclado TTY).
4. Cede la ejecución (`yield`) al menú que lo haya llamado.

---

## 5. API de Interfaces y Menús

### `interactive_menu_select(...) -> int`
La joya de la corona del módulo. Renderiza un menú dinámico en pantalla completa con múltiples capas de personalización.
- Soporta sub-títulos, arte ASCII inyectable y esquemas ("fantasy" o "default").
- Soporta modo "Buffer": Si el jugador aprieta `1` y luego `5`, la lógica detiene momentáneamente la navegación con flechas y muestra el buffer `15`, saltando a la opción en cuanto se presione Enter.
- Utiliza la clase `Live` para inyectar frames a 60FPS sin generar logs residuales en la consola y `transient=True` no está activado por defecto, permitiendo que la terminal limpie su bloque solo al usar un `.screen()` superior.

### `select_option_with_arrows(prompt, valid_options) -> int`
Versión de bajo consumo para diálogos en línea (inline-prompts).
Imprime un `\r` (Carriage Return) para sobreescribir la misma línea en la terminal, lo que permite navegar por las opciones (Ej: `[1] Si / [2] No`) usando las flechas de manera rápida sin inicializar todo el andamiaje `rich.Live`.

### `wait_for_enter(Message=None) -> None`
Pausa absoluta de ejecución. Secuestra la terminal obligando al usuario a confirmar un texto antes de continuar el Main Loop.

---

## 6. Flujos Críticos e Interacción Asíncrona

Casi toda función de Menú implementada en este motor utiliza el siguiente patrón dictado por `input_utils.py`:
1. El programador instancia un diccionario atado (`state = {"idx": 0, "done": False, "dirty": True}`).
2. Declara un `def _on_press(key):` que mutará el diccionario padre mediante inyección de eventos desde `listen_keyboard`.
3. Envuelve la ejecución en un hilo Demonio (`daemon=True`) que muere al cerrarse la app.
4. El hilo principal (`main_thread`) mantiene secuestrado el motor en un bucle `while not state["done"]:`, consultando el flag `"dirty"` a muy alta velocidad (`time.sleep(0.01)`). Si el flag es afirmativo, redibuja. Si no, aguarda, ahorrando un ~80% de recursos de CPU en interfaces pausadas.
