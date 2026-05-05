# `Loader.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Función `Read()`](#función-read)
3. [Función `Write()`](#función-write)
4. [Manejadores Especializados (Handlers)](#manejadores-especializados-handlers)
5. [Sistema de Logs](#sistema-de-logs)
6. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`Loader.py` es el sistema de persistencia y acceso a datos centralizado del juego. Proporciona una capa de abstracción sobre el sistema de archivos, permitiendo leer y escribir configuraciones complejas en formato JSON y logs en texto plano con validación automática y manejo de errores.

---

## Función `Read()`

```python
Read(FilePath, Object, Content=None, Values=None)
```
Busca y extrae información de un archivo JSON.
- **FilePath**: Ruta al archivo.
- **Object**: Clave principal (ej. nombre de una zona o categoría).
- **Content**: Sub-clave opcional para acceder a datos anidados.

### Comportamiento Inteligente
La función detecta automáticamente el tipo de archivo por su nombre (ej. `datastats.json`) y aplica una lógica de extracción específica para navegar por estructuras de listas o diccionarios complejos.

---

## Función `Write()`

```python
Write(FilePath, Object=None, Content=None, ToWrite=None, isplaintext=False)
```
Guarda datos en disco.
- **JSON**: Si `isplaintext=False`, actualiza el archivo JSON manteniendo la estructura existente (no sobrescribe todo el archivo, solo la sección indicada).
- **Plain Text**: Si `isplaintext=True`, añade una línea al final del archivo (útil para logs).

---

## Manejadores Especializados (Handlers)

El sistema incluye lógica interna para tratar archivos con estructuras no convencionales:

- **`_handle_datastats`**: Maneja zonas que contienen arrays de enemigos y una sección especial de `Strings`.
- **`_handle_weapons`**: Soporta el prefijo `TYPE:` en las claves de armas.
- **`_handle_buffs`**: Diferencia entre las categorías `BUFFS` y `DEBUFFS`.
- **`_handle_saves`**: Preserva valores nulos para asegurar que el estado del jugador se guarde correctamente.

---

## Sistema de Logs

Utiliza el módulo `logging` de Python para escribir en `tpprpg.log`.
- Registra advertencias cuando una clave no existe.
- Registra errores críticos de sintaxis JSON o archivos no encontrados.
- Muestra mensajes visuales en la consola mediante `rich` para informar al desarrollador.

---

## Ejemplos de Uso

### Leer los enemigos de una zona

```python
from Modules.ModulesUtils.Loader import Read

# Retorna la lista de enemigos en la zona "Forest"
enemigos = Read("./Data/DataStats.json", "Forest")
```

### Guardar el progreso en un slot

```python
from Modules.ModulesUtils.Loader import Write

# Guarda el nivel del jugador en el Slot 1
Write("./Data/SaveSlots.json", "Save1", "Level", 10)
```

### Escribir un log personalizado

```python
Write("./logs/debug.txt", Content="El jugador entró en combate", isplaintext=True)
```
---

## Notas Técnicas

- Utiliza `os.path.normpath` para asegurar compatibilidad entre Windows y Linux.
- Implementa codificación `utf-8` en todas las operaciones de E/S.
- El sistema de escritura crea automáticamente los directorios padre si no existen.
