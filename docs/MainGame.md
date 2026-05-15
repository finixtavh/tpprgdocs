# `MainGame.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [El Bucle Principal (Main Loop)](#bucle-principal)
4. [Menú Principal e Interacción](#menu-principal)
5. [Sistema de Combate Detallado](#sistema-combate)
6. [Carga Procedural y Zonas](#carga-zonas)

---

## 1. Descripción General
`MainGame.py` es el orquestador principal del motor TPPRPG. Contiene el bucle de juego interactivo (`main()`), donde el jugador invierte la mayor parte de su tiempo navegando opciones, visualizando su `Dashboard` de estado (HP/MP/Stats) e interactuando con otros módulos a través del enrutador del menú principal. También actúa como el gestor gráfico del sistema de combate, renderizando los intercambios de daño entre entidades.

---

## 2. Dependencias e Inyecciones
- **Archivos Base**: Su núcleo depende de prácticamente toda la base de código (`Modules.ModulesManager`, `Modules.ModulesFunctions`, `Modules.ModulesMenu`, y `Modules.ModulesUtils`).
- **Librerías TUI**:
  - `rich.console`, `rich.panel`, `rich.table`, `rich.text`, `rich.live`, `rich.align`.
  - Escucha de eventos de teclado asíncronos mediante `sshkeyboard`.
- **Inyección de Dependencias (DI)**: Inicializa e inyecta Singleton Managers (`ItemManager`, `ShopManager`, `BuffManager`, etc.) a los diferentes sub-menús cuando el jugador los selecciona.

---

## 3. El Bucle Principal (Main Loop)
El juego no se apaga a menos que el jugador seleccione "0" en el menú principal. Su estructura funciona así:
1. Limpia la pantalla `os.system('cls'/'clear')`.
2. Llama a `player.Check(vInventory)` pasivamente para actualizar buffs temporales y estado de salud.
3. Imprime el Dashboard en vivo invocando a `display_player_status()`.
4. Rinde el control al menú interactivo llamando a `display_main_menu()`.
5. Procesa el `main_input` devuelto en un árbol masivo de `if / elif`, ejecutando los flujos correspondientes.

---

## 4. Menú Principal e Interacción

El menú interactivo provee un router hacia 16 subsistemas y 1 menú extra oculto:
- `[1-2]` **Zonas**: Iniciar exploración o combate con el gestor de zonas.
- `[3]` **Tiendas**: Inicializa `ShopManager`.
- `[4]` **Inventario**: Delega a `InventoryMenu.py`.
- `[5]` **Estadísticas**: Un renderizado nativo multi-pestaña usando flechas que expone stats puras de combate, maestría e información de aliados activos.
- `[6]` **Gathering**: Dispara un minijuego de farmeo usando `GatheringMasterySystem.py`.
- `[7-9]` **Mods, Items y Guardado**: Interfaces utilitarias y de inyección JSON externa.
- `[10-16]` **Progresión de Jugador y Geopolítica**: Árboles de habilidades, mesa de encantamientos, `BaseSystem` de jugador individual y `GuildMenu` geopolítico.
- `[17] DEBUG**: (Solo si `CONFIG["DEBUG"]["ENABLED"] == True`) Abre consola técnica.

---

## 5. Sistema de Combate Detallado

Cuando un jugador inicia una pelea, el flujo cae en `combat_loop()` (o lógicas similares inyectadas):
1. **Dibuja el estado (Render de Pantalla)**: Llama a `render_combat_menu()`. Enumera los componentes `HP/MP/Barrera` del jugador, de sus compañeros y del enemigo. Renderiza un bloque de penalizaciones/bonos climáticos globales.
2. **Ciclo de Reacciones**: `_perform_attack(attacker, defender)` ejecuta daños y lee si hubo "Evasión".
3. **Engine de Cálculos Visual**: Llama a `_display_combat_details()`. Desglosa el ataque real (que es un diccionario complejo devuelto por `Entity.attack()`) en barras y paneles coloreados desglosando Físico/Magia e imprimiendo los descuentos de Defensa Plana, Encantamientos y Barreras que mitigaron el golpe.
4. **Contraataque Asíncrono**: Si el diccionario retornado tiene la flag `is_counter = True`, el sistema detiene temporalmente la ejecución para dibujar un ataque relámpago de respuesta fuera de turno antes de seguir.

---

## 6. Carga Procedural y Zonas
La función `zone_loader(player_zone)` encapsula la lógica para instanciar a los monstruos con los que el jugador va a pelear:
- Busca si algún mod cargó en memoria los datos para esa zona (`mod_api.stats_injections`). Si no, recurre a `Data/DataStats.json`.
- Filtra del Array base de criaturas al jefe final (`IS_BOSS`) usando `SPAWN_CHANCE`. Si se superó el RNG, reemplaza el pool y fuerza un combate crítico; de lo contrario, elige aleatoriamente un esbirro estándar del JSON.
- Una variante (`zone_loader_group`) permite generar arenas de combate con N cantidad de oponentes simultáneos, útil para misiones asíncronas de base o incursiones (Raids).
