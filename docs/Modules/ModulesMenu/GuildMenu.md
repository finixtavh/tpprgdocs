# `GuildMenu.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Estructura de la Interfaz (Sub-Menús)](#sub-menus)
5. [Funciones del Módulo (API)](#funciones)
6. [Manejo de Renderizado y Teclado](#renderizado)

---

## 1. Descripción General
`GuildMenu.py` contiene la interfaz visual del sistema Geopolítico (`GuildSystem.py`). Implementa una suite de navegación basada en Live/Panels mediante la librería `rich`. A través de sub-menús interactivos, el jugador administra los territorios conquistados, sus finanzas y recursos, interactúa diplomáticamente declarando la guerra/paz, realiza sabotajes asíncronos y dirige proyectos de construcción.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Requiere el `GuildManager` Singleton, lee la lista constante de edificios `GUILD_BUILDINGS` y se inyecta indirectamente con `config.json` para extraer las métricas punitivas por acciones bélicas (`FORMAL_WAR_PENALTY`, `SURPRISE_WAR_PENALTY`).
- **Librerías TUI**:
  - `rich.console`, `rich.panel`, `rich.table`, `rich.live`.
  - `sshkeyboard` inyectado dentro del decorador/context manager `keyboard_listener_scope`.
- **Módulos Conectados**: Lanza a su vez sub-menús como `open_guild_skill_tree` y `open_guild_missions_menu`.

---

## 3. Constantes y Variables Globales
- `RARITY_COLORS` (`Dict`): Un mapeo simple que asigna colores HEX o string de la TUI según la calidad del material de guild (`Common` a `Legendary`).

---

## 4. Estructura de la Interfaz (Sub-Menús)

El menú principal delega la pantalla a 6 paneles distintos (Pestañas):
1. **Resumen**: Estadísticas maestras (Moral, Poder Militar, Reputación del Jugador, y Cola de Últimos Eventos).
2. **Territorios**: Mapa ASCII y listado de todas las zonas que tiene el jugador (`get_territory_map_rows()`).
3. **Mundo**: Muestra a la competencia. Lista todos los clanes ordenador por Poder Militar (`get_sorted_guilds_by_power()`).
4. **Diplomacia**: Permite Aliarse (`[F]`), Declarar Guerra (`[W]`), Sabotear (`[S]`) u Ofrecer Tratos Comerciales (`[T]`).
5. **Recursos**: Estado de stock, capacidad máxima y consumo/ticks restantes de alimento.
6. **Edificios**: Cola de construcción de la base.

---

## 5. Funciones del Módulo (API)

### Renderizadores "Stateless"
Solo devuelven strings o paneles. No bloquean.
- `_render_summary(guild, player, gm)`
- `_render_territories(guild, gm, page)`
- `_render_resources(guild, gm)`
- `_render_world(gm, player, page)`
- `_render_diplomacy(guild, gm, player, page)`

### Hooks de Controladores de Sub-Menús
Bloqueantes (Bucle Atrapado). Toman el control de la terminal para realizar acciones de gestión que cambian los números del juego.
- `_diplomacy_action_menu(player_guild, player, gm, con)`
- `_territories_action_menu(player_guild, player, gm, con)`
- `_buildings_menu(player_guild, gm, con)`
- `_events_menu(player_guild, player, gm, con)`: Ejecuta las colas de decisión y lee los inputs procedimentales de las misiones que saltan cada N minutos.
- `_custom_trade_menu(player_guild, target_guild, player, gm, con)`: Sistema de trueque con la IA.

### Punto de Entrada
- `open_guild_menu(player, console=None)`
  - Encierra al usuario en una brújula principal con las flechas Izquierda/Derecha rotando entre las pantallas. Si se apreta `ENTER`, invoca a la función controladora de la Pestaña actual y el sub-menú toma temporalmente el Thread de teclado.

---

## 6. Manejo de Renderizado y Teclado

### La Regla de Oro del Renderizado en `GuildMenu.py`:
A diferencia de otros submenús simples, el `GuildMenu` nunca anida un `Live` dentro de otro `Live`.
Cada controlador (`_buildings_menu`, `_diplomacy_action_menu`, etc.) sigue este patrón estructurado:
```python
with con.screen():
    with keyboard_listener_scope():
        kb_thread = threading.Thread(target=listen_keyboard, kwargs={...}, daemon=True)
        kb_thread.start()
        with Live(console=con, screen=True, auto_refresh=False) as live:
            try:
                while not state["exit"]:
                    if state["dirty"]:
                        live.update(_render(), refresh=True)
                        state["dirty"] = False
                    time.sleep(0.01)
            finally:
                stop_listening()
                kb_thread.join(timeout=0.2)
        con.clear()
```
Esto asegura que la terminal devuelva el foco limpiamente y no arroje *fantasmas* (líneas basura en la consola) ni bugs de teclas atascadas (debido a colisiones de threads de `sshkeyboard`) al regresar a la vista anterior.
