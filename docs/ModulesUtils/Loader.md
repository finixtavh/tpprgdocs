# `Loader.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Configuración de Logging](#configuración-de-logging)
3. [Función `Read`](#función-read)
4. [Función `Write`](#función-write)
5. [Handlers internos de lectura](#handlers-internos-de-lectura)
6. [Handlers internos de escritura](#handlers-internos-de-escritura)
7. [Ejemplos de Uso](#ejemplos-de-uso)
8. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`Loader.py` es el módulo de **entrada/salida de datos** del juego. Proporciona dos funciones principales (`Read` y `Write`) que abstraen la lectura y escritura de archivos JSON con manejo de errores robusto, logging automático y soporte para múltiples estructuras de archivo.

Detecta automáticamente el tipo de archivo por nombre y aplica la lógica de parseo correspondiente.

---

## Configuración de Logging

Al importarse, el módulo configura un logger que guarda eventos en `tpprpg.log`:

```python
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    filename='tpprpg.log',
    filemode='a'
)
logger = logging.getLogger('TPPRPG.Loader')
```

Todos los errores de lectura/escritura quedan registrados automáticamente con timestamp, nivel y mensaje de excepción.

---

## Función `Read`

```python
Read(FilePath, Object, Content=None, Values=None)
```

Lee y procesa datos JSON desde archivos del juego. Detecta el tipo de archivo por su nombre y aplica el handler correspondiente.

### Parámetros

| Parámetro | Tipo | Descripción |
|---|---|---|
| `FilePath` | `str` | Ruta al archivo JSON. Se normaliza con `os.path.normpath`. |
| `Object` | `str` o `None` | Clave primaria a acceder en el JSON. Si es `None`, retorna el JSON completo. |
| `Content` | `str` o `None` | Clave secundaria o comando especial (`"Strings"`, `"IDs"`, etc.). |
| `Values` | `any` | Parámetro adicional reservado para consultas más complejas (no usado actualmente). |

### Retorno

Retorna la estructura de datos solicitada. Si ocurre un error (archivo no encontrado, JSON inválido, clave inexistente), retorna una lista vacía `[]` o diccionario vacío `{}` para consistencia.

### Detección de tipo de archivo

| Nombre del archivo | Handler aplicado |
|---|---|
| `datastats.json` | `_handle_datastats` |
| `weapons.json` | `_handle_weapons` |
| `datashops.json` | `_handle_shops` |
| `saveslots.json` / `savegame.json` | `_handle_saves` |
| `databuffs.json` | `_handle_buffs` |
| Cualquier otro | `_handle_generic_json` |

### Manejo de errores

| Excepción | Acción |
|---|---|
| `FileNotFoundError` | Imprime error, registra en log, retorna `[]`. |
| `json.JSONDecodeError` | Imprime error de JSON malformado, retorna `[]`. |
| `Exception` genérica | Captura cualquier otro error, retorna `[]`. |

---

## Función `Write`

```python
Write(FilePath, Object=None, Content=None, ToWrite=None, isplaintext=False)
```

Escribe datos en archivos JSON o texto plano.

### Parámetros

| Parámetro | Tipo | Descripción |
|---|---|---|
| `FilePath` | `str` | Ruta del archivo a escribir. |
| `Object` | `str` o `None` | Clave primaria en el JSON donde escribir. |
| `Content` | `str` o `None` | Clave secundaria o contenido (texto plano). |
| `ToWrite` | `any` | Datos a escribir en el JSON. |
| `isplaintext` | `bool` | Si `True`, escribe como texto plano en modo *append*. |

### Comportamiento

**Texto plano** (`isplaintext=True`): Crea el directorio si no existe y añade `Content + "\n"` al final del archivo.

**JSON** (`isplaintext=False`):

1. Lee el archivo existente (o crea un dict vacío si no existe).
2. Aplica el handler del tipo de archivo para actualizar la estructura.
3. Escribe el JSON resultante con `indent=4`.

---

## Handlers internos de lectura

### `_handle_datastats(data, Object, Content)`

Para `DataStats.json`. Maneja zonas con enemigos y strings de descripción.

| `Content` | Retorna |
|---|---|
| `None` | Todos los datos de la zona. |
| `"Strings"` | El objeto `Strings` de la zona (nombre, descripción, color). |
| Nombre de enemigo | Los datos del enemigo específico. |

---

### `_handle_weapons(data, Object, Content)`

Para `weapons.json`. Maneja tipos de armas con prefijo `"TYPE:"`.

| `Content` | Retorna |
|---|---|
| `None` | Todas las armas del tipo. |
| Nombre de arma | Los datos del arma específica. |

Añade automáticamente el prefijo `"TYPE:"` si el `Object` no lo tiene.

---

### `_handle_shops(data, Object, Content)`

Para `DataShops.json`. Maneja tiendas con arrays de IDs.

| `Content` | Retorna |
|---|---|
| `None` | Todos los datos de la tienda. |
| `"IDs"` | Solo el array de IDs de items. |

---

### `_handle_saves(data, SaveSlot, Content)`

Para `SaveSlots.json` / `SaveGame.json`. Preserva valores `null` intencionales.

| `Content` | Retorna |
|---|---|
| `None` | Todos los datos del slot de guardado. |
| Clave específica | Valor del campo (puede ser `None`). |

---

### `_handle_buffs(data, Object, Content)`

Para `DataBuffs.json`. Maneja buffs y debuffs.

| `Object` | `Content` | Retorna |
|---|---|---|
| `"BUFFS"` o `"DEBUFFS"` | `None` | Todos los buffs/debuffs de esa categoría. |
| `"BUFFS"` o `"DEBUFFS"` | ID específico | El buff/debuff con ese ID. |
| ID con `"_buff"` o `"_debuff"` | — | El buff/debuff directamente. |

---

### `_handle_generic_json(data, Object, Content)`

Handler genérico para archivos sin estructura especial. Busca `Content` en el objeto, incluyendo dentro de listas.

---

## Handlers internos de escritura

Cada handler de escritura espeja la estructura de su handler de lectura para mantener la integridad del JSON:

| Handler | Archivo objetivo |
|---|---|
| `_write_datastats` | `DataStats.json` |
| `_write_weapons` | `weapons.json` |
| `_write_shops` | `DataShops.json` |
| `_write_saves` | `SaveSlots.json` / `SaveGame.json` |
| `_write_buffs` | `DataBuffs.json` |
| `_write_generic_json` | Cualquier otro JSON |

---

## Ejemplos de Uso

### Leer todos los enemigos de una zona

```python
from Modules.Loader import Read

enemies_data = Read("Data/DataStats.json", "Forest", None)
# Retorna la lista completa de datos del Bosque
```

### Leer las strings (nombre, descripción) de una zona

```python
strings = Read("Data/DataStats.json", "Forest", "Strings")
name = strings.get("name", "Bosque")
desc = strings.get("desc", "Sin descripción")
color = strings.get("color", "white")
```

### Leer datos completos del save

```python
save_data = Read("Data/SaveGame.json", None, None)
# Retorna todo el JSON
```

### Guardar datos del jugador

```python
from Modules.Loader import Write

Write("Data/SaveGame.json", "player", "gold", 9999)
```

### Escribir un log en texto plano

```python
Write("logs/logs.txt", None, "[INFO] El jugador subió de nivel.", None, isplaintext=True)
```

---

## Notas y Referencias

- La función `Read` siempre retorna un valor seguro (lista o dict vacío) en caso de error, lo que previene que el juego crashee por problemas de archivo.
- **`os.path.normpath`:** Convierte separadores de ruta al formato del SO actual (`/` en Linux, `\` en Windows). [https://docs.python.org/3/library/os.path.html#os.path.normpath](https://docs.python.org/3/library/os.path.html#os.path.normpath)
- **`logging` en Python:** [https://docs.python.org/3/library/logging.html](https://docs.python.org/3/library/logging.html)
- **`json.load` vs `json.loads`:** `json.load` lee desde un archivo, `json.loads` desde un string. [https://docs.python.org/3/library/json.html](https://docs.python.org/3/library/json.html)
- **`pathlib.Path.mkdir(parents=True, exist_ok=True)`:** Crea directorios anidados sin errores si ya existen. [https://docs.python.org/3/library/pathlib.html#pathlib.Path.mkdir](https://docs.python.org/3/library/pathlib.html#pathlib.Path.mkdir)
