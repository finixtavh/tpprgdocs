# `GuildMenu.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Estructura de la Interfaz](#estructura-de-la-interfaz)
3. [Módulos del Menú](#módulos-del-menú)
    - [Resumen de Guild](#resumen-de-guild)
    - [Gestión de Territorios](#gestión-de-territorios)
    - [Diplomacia y Guerra](#diplomacia-y-guerra)
    - [Economía y Recursos](#economía-y-recursos)
    - [Construcciones](#construcciones)
4. [Navegación y Controles](#navegación-y-controles)
5. [Sistema de Renderizado (Rich + Live)](#sistema-de-renderizado-rich--live)

---

## Descripción General

`GuildMenu.py` es la interfaz maestra para el sistema de simulación de gremios. Proporciona al jugador una visión estratégica del mundo, permitiéndole gestionar su propia guild, interactuar diplomáticamente con otras facciones, expandir su territorio y desarrollar infraestructuras.

---

## Estructura de la Interfaz

La interfaz utiliza un sistema de pestañas virtuales y pantallas dedicadas:
1. **Pestaña Principal**: Resumen de poder, moral y población.
2. **Mapa de Territorios**: Visualización ASCII del control territorial del mundo.
3. **Panel Diplomático**: Relaciones, alianzas, estados de guerra y **Sabotaje**.
4. **Mercado y Trueques**: Gestión de propuestas comerciales de otras guilds.
5. **Almacén de Recursos**: Stock de materiales y comercio de excedentes.
6. **Menú de Construcción**: Desarrollo de edificios que otorgan bonus pasivos.
7. **Configuración de Taxes**: Slider para ajustar el % de impuestos (1-50%).

---

## Módulos del Menú

### Resumen de Guild
Muestra las estadísticas vitales:
- **Poder Militar**: Fuerza de ataque y defensa en conquistas.
- **Moral**: Afecta la eficiencia y puede provocar deserciones si es baja.
- **Influencia**: Puntos para acciones diplomáticas y políticas.
- **Reputación del Jugador**: Cómo ve la guild al jugador específicamente.

### Gestión de Territorios
Permite visualizar el mapa del mundo:
- **★**: Territorio propio.
- **🔸**: Territorio de otra guild.
- **·**: Territorio neutral.
- **Acción**: El jugador puede declarar la guerra o atacar territorios neutrales consumiendo recursos (Lingotes de Xar, Madera).

### Diplomacia y Guerra
Permite interacciones complejas con otras facciones:
- **Alianzas**: Cooperación militar y comercial.
- **Guerra Formal**: Conflicto declarado que aumenta la **Tensión Mundial** y permite la conquista.
- **Sabotaje**: Requiere la habilidad *Espionaje*. Permite robar oro o dañar la moral enemiga.

### Economía y Recursos
Visualiza el flujo de materiales:
- **Consumo de Comida**: Calculado por tick según la población.
- **Generación de Recursos**: Proveniente de los territorios controlados.
- **Capacidad**: Limitada por el tipo de guild y edificios construidos.

### Construcciones
Lista de edificios disponibles (Muro, Cuartel, Almacén, etc.) con sus costes en materiales de guild y efectos inmediatos sobre las estadísticas del gremio.

---

## Navegación y Controles

| Tecla | Acción |
|---|---|
| **↑ / ↓** | Navegar entre opciones de menú o listas de territorios. |
| **← / →** | Cambiar entre pestañas principales o páginas de listas. |
| **Enter** | Confirmar acción (Construir, Atacar, Aliar). |
| **F** | Formar Alianza (en el menú de Diplomacia). |
| **W** | Declarar Guerra (en el menú de Diplomacia). |
| **S** | Sabotear (requiere Espionaje). |
| **T** | Comercio Personalizado con NPC. |
| **O** | Abrir Configuración de Taxes. |
| **Esc** | Volver o cerrar el menú. |

---

## Sistema de Renderizado (Rich + Live)

El menú utiliza `rich.live` para actualizaciones en tiempo real (20-60 FPS), permitiendo que los eventos del mundo (notificaciones de otras guilds, cambios de clima) se reflejen instantáneamente sin parpadeos de pantalla. Cada submenú se encapsula en un `console.screen()` para garantizar una limpieza total de la terminal al salir.

---

## Notas Técnicas

- La configuración de costes y penalizaciones se lee dinámicamente de `config.json`.
- El sistema de mapa ASCII se genera proceduralmente a través del `GuildManager`.
- La diplomacia incluye un sistema de "Reputación Global" que afecta cómo te ven todas las guilds si realizas acciones deshonestas (como guerras sorpresa).
