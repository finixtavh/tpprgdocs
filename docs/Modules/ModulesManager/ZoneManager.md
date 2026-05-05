# `ZoneManager.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Gestor: `ZoneManager`](#gestor-zonemanager)
3. [Estructura de Datos de Zona](#estructura-de-datos-de-zona)
4. [Integración con Mods](#integración-con-mods)
5. [Enemigos y Recursos (Gathering)](#enemigos-y-recursos-gathering)
6. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`ZoneManager.py` es el encargado de indexar y proporcionar información sobre las distintas áreas geográficas del juego. Su función principal es leer `DataStats.json` (donde residen los datos de enemigos y loot) y extraer las metadatos de las zonas para que el jugador pueda seleccionarlas y viajar a ellas.

---

## Gestor: `ZoneManager`

Clase centralizada que escanea tanto los archivos base como las inyecciones de mods para construir el mapa del mundo.

### Funciones Principales
- `get_all_zones()`: Consolida todas las zonas (Base + Mods) en un solo diccionario.
- `get_zones_list()`: Retorna una lista simplificada diseñada para alimentar menús de selección visual.
- `get_zone_enemies(zone_key)`: Filtra los datos de la zona para extraer exclusivamente la lista de combatientes posibles.

---

## Estructura de Datos de Zona

Cada zona en `DataStats.json` debe contener una sección `Strings` para ser reconocida por el gestor:

- `name`: Nombre legible (ej. "Bosque de los Lamentos").
- `desc`: Breve descripción ambiental.
- `color`: Color de representación en la UI (Rich).
- `exit`: Mensaje mostrado al abandonar el área.

---

## Integración con Mods

El `ZoneManager` consulta activamente al `ModLoader`. Si un mod inyecta nuevas zonas, estas aparecerán automáticamente en el selector de viajes sin necesidad de modificar el código fuente original.

---

## Enemigos y Recursos (Gathering)

Además de los metadatos visuales, el gestor extrae:
1. **Enemigos**: Identifica objetos que contienen el campo `NAME` (excluyendo secciones especiales).
2. **Gathering**: Mapea los recursos recolectables asociados a la zona para que el sistema de recolección sepa qué materiales hay disponibles.

---

## Ejemplos de Uso

### Obtener lista de zonas para un menú interactivo

```python
from Modules.ModulesManager.ZoneManager import get_zones_for_selector

# Retorna [(zone_key, name, color), ...]
zonas = get_zones_for_selector()
for key, name, color in zonas:
    print(f"Viajar a {name}")
```

### Consultar enemigos de la zona actual

```python
from Modules.ModulesManager.ZoneManager import get_zone_enemies

# enemies será una lista de dicts con stats de monstruos
enemies = get_zone_enemies("ShadowRealm")
```
---

## Notas Técnicas

- Utiliza el patrón **Singleton** para evitar lecturas repetidas de disco.
- Filtra automáticamente secciones administrativas como `Strings` y `Gathering` para separar la lógica de combate de la de metadatos.
