# `DebugMenu.py`

## Índice

1. [Descripción General](#descripción-general)
2. [El Menú Debug Mejorado](#el-menú-debug-mejorado)
3. [Funciones de Trampa (Cheats)](#funciones-de-trampa-cheats)
4. [Visor de Variables](#visor-de-variables)
5. [Integración con Sistemas](#integración-con-sistemas)
6. [Controles](#controles)

---

## Descripción General

`DebugMenu.py` proporciona un conjunto de herramientas de desarrollo accesibles durante la ejecución del juego. Permite a los desarrolladores y testers manipular el estado del mundo, el jugador y los sistemas subyacentes sin necesidad de editar archivos de guardado o reiniciar el juego.

---

## El Menú Debug Mejorado

### `open_enhanced_debug_menu(...)`
Una interfaz visual basada en `Rich` que ofrece una lista de acciones rápidas. Utiliza un bucle de eventos con `sshkeyboard` para una navegación fluida.

---

## Funciones de Trampa (Cheats)

El menú incluye las siguientes capacidades:
- **Max Recursos**: Añade 9,999,999 de Oro y EXP al jugador.
- **Boost Mastery**: Incrementa en +20 niveles todas las habilidades de recolección y combate (Mastery).
- **Guild SP**: Otorga +20 puntos de habilidad a todos los gremios del mundo.
- **Añadir Armas**: Inyecta equipamiento de alto nivel directamente en el inventario.
- **Compañero de Prueba**: Crea instantáneamente un compañero "Lyra (Debug)" para pruebas de combate.
- **Provocar Desastre**: Fuerza al `WeatherSystem` a generar un evento climático catastrófico.
- **Buff/Debuff Aleatorio**: Aplica estados alterados para probar mecánicas de combate.

---

## Visor de Variables

### `open_variable_viewer(...)`
Una herramienta de diagnóstico dividida por pestañas:
- **Jugador**: Nombre, nivel, oro, ítems descubiertos, recetas favoritas.
- **Stats**: HP/MP actuales, regeneración, defensas elementales, evasión, contrataque.
- **Mastery**: Desglose exacto de niveles y EXP en cada disciplina.
- **Mundo**: Zona actual, cantidad de compañeros activos, estado de la base.
- **Sistema**: Cantidad de mods cargados y elementos inyectados por la API.

---

## Integración con Sistemas

El menú debug actúa como un puente entre múltiples módulos:
- **`ItemManager`**: Para la creación de objetos.
- **`WeatherSystem`**: Para el control del clima.
- **`GuildSystem`**: Para la manipulación de gremios.
- **`BuffManager`**: Para la gestión de estados.

---

## Controles

| Tecla | Acción |
|---|---|
| **Flecha Arriba/Abajo** | Navegar entre opciones o pestañas. |
| **Enter** | Ejecutar acción seleccionada. |
| **Esc** | Salir del menú o volver a la pantalla anterior. |

---

## Notas Técnicas

- Utiliza `console.screen()` para asegurar que la terminal se limpie completamente al abrir y cerrar el menú, evitando artefactos visuales.
- El visor de variables es de solo lectura y se actualiza en tiempo real (15 FPS).
