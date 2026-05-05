# `MainGame.py`

## Índice

1. [Descripción General](#descripción-general)
2. [El Bucle Principal (Main Loop)](#el-bucle-principal-main-loop)
3. [Sistema de Combate Detallado](#sistema-de-combate-detallado)
    - [Ataques y Contraataques](#ataques-y-contraataques)
    - [Visualización de Resultados](#visualización-de-resultados)
4. [Gestión de Zonas y Enemigos](#gestión-de-zonas-y-enemigos)
5. [Dashboard del Jugador](#dashboard-del-jugador)
6. [Opciones del Menú Principal](#opciones-del-menú-principal)
7. [Configuración y Debug](#configuración-y-debug)

---

## Descripción General

`MainGame.py` es el punto de entrada y el orquestador central de TPPRPG. Importa todos los gestores y sistemas funcionales para unificar la experiencia de juego, manejando la transición entre la exploración, el combate, el comercio y la gestión de gremios.

---

## El Bucle Principal (Main Loop)

El juego opera bajo un bucle `while True` que presenta al jugador el **Menú Principal**. Cada acción seleccionada dispara una función específica que puede abrir otros submenús interactivos o iniciar una secuencia de combate. 

**Mecánicas de Ciclo:**
- **player.Check()**: Se llama tras cada acción para actualizar estados y buffs.
- **fight_count**: Se incrementa tras cada combate finalizado. Al llegar a **5**, se activa automáticamente la recaudación de impuestos de la Guild del jugador (ver `GuildSystem.py`).
- **Global Tick**: El sistema de gremios y clima avanza sus estados internos tras acciones de exploración.

---

## Sistema de Combate Detallado

### Ataques y Contraataques
El combate es por turnos pero incluye mecánicas de reacción:
- **Evasión**: El defensor tiene una probabilidad de evitar todo el daño.
- **Contraataque**: El defensor puede responder con un ataque inmediato sin consumir su turno si sobrevive al golpe.
- **Cadena**: Los ataques pueden involucrar a compañeros, creando combates de grupo.

### Visualización de Resultados
La función `_display_combat_details` utiliza `Rich` para renderizar un desglose matemático de cada golpe:
- **Daño por Tipo**: Barras de colores para Físico, Térmico, Tierra, Eléctrico y Profundo.
- **Descuentos**: Muestra cuánto daño fue absorbido por la Defensa Plana, la Defensa Porcentual, los Encantamientos y la Barrera.
- **Modificadores Ambientales**: Indica cómo el clima actual afectó la potencia del ataque.

---

## Gestión de Zonas y Enemigos

### `zone_loader()`
Carga dinámicamente el contenido de un área:
1. Consulta si los **Mods** han inyectado datos para esa zona.
2. Si no, carga desde el `DataStats.json` base.
3. Filtra jefes finales (Bosses) basándose en su `SPAWN_CHANCE`.
4. Devuelve un enemigo o grupo de enemigos para iniciar el encuentro.

---

## Dashboard del Jugador

Muestra la información vital en la parte superior de la terminal:
- **Barras de Estado**: HP (con overlay de Barrera cian) y MP.
- **Info de Mundo**: Zona actual, Clima activo y Oro acumulado.
- **Nivel**: Nivel actual del jugador y progreso de experiencia.

---

## Opciones del Menú Principal

| ID | Opción | Descripción |
|---|---|---|
| 1 | Inspeccionar zonas | Ver enemigos y recursos de cualquier área. |
| 2 | Ir a una zona | Viajar e iniciar un encuentro de combate. |
| 3 | Tienda | Comprar equipamiento y consumibles. |
| 4 | Inventario | Gestionar ítems y equipar objetos. |
| 11 | Habilidades | Abrir el árbol de talentos del jugador. |
| 14 | Base | Gestionar edificios y minijuegos de la base. |
| 16 | Guilds | Acceder a la simulación geopolítica de gremios. |

---

## Configuración y Debug

- **config.json**: El juego lee parámetros como la duración de las barras, el multiplicador de dificultad y el estado del modo Debug.
- **Modo Debug**: Si está activado en la configuración, aparece una opción adicional (17) que da acceso al `DebugMenu.py`.
- **Logging**: Captura errores inesperados y los guarda en `tpprpg.log` junto con un snapshot del estado de las variables.
