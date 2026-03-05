# `check_admin.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Variables y Constantes](#variables-y-constantes)
3. [Funciones](#funciones)
4. [Cómo funciona](#cómo-funciona)
5. [Ejemplos de Uso](#ejemplos-de-uso)
6. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

`check_admin.py` es un módulo utilitario que verifica si el juego se está ejecutando con **privilegios de administrador** (Windows) o **root** (Linux/macOS), y si no los tiene, los solicita reiniciando el proceso con los permisos necesarios.

Es cross-platform: detecta automáticamente el sistema operativo y usa el método apropiado para cada uno.

---

## Variables y Constantes

| Variable | Tipo | Descripción |
|---|---|---|
| `PROJECT_ROOT` | `Path` | Ruta absoluta a la raíz del proyecto. Se calcula a partir de la ubicación del propio archivo (`__file__`). |
| `LOG_PATH` | `Path` | Ruta al archivo de logs: `PROJECT_ROOT/logs/logs.txt`. |

```python
PROJECT_ROOT = Path(__file__).resolve().parent.parent
LOG_PATH = PROJECT_ROOT / "logs" / "logs.txt"
```

`PROJECT_ROOT` también se agrega a `sys.path` si no está presente, para asegurar que los imports del proyecto funcionen correctamente sin importar desde dónde se ejecute el script.

---

## Funciones

### `is_admin() -> bool`

Detecta si el proceso actual tiene privilegios elevados.

**Comportamiento según SO:**

| Sistema | Método |
|---|---|
| **Windows** | Usa `ctypes.windll.shell32.IsUserAnAdmin()`. Retorna `True` si el valor es distinto de 0. |
| **Linux / macOS** | Comprueba si `os.geteuid() == 0` (el UID 0 corresponde a root). |

Retorna `True` si hay privilegios, `False` si no.

```python
if is_admin():
    print("Ejecutando como administrador")
```

---

### `request_admin() -> None`

Reinicia el proceso actual solicitando privilegios elevados. **No retorna nada útil**; en su lugar, reemplaza o relanza el proceso.

**Comportamiento según SO:**

**Windows:**

Usa `ctypes.windll.shell32.ShellExecuteW` con el verbo `"runas"` para abrir un diálogo de UAC (User Account Control) que solicita permisos de administrador. Luego termina el proceso actual con `sys.exit()`.

**Linux / macOS:**

Usa `os.execvp("sudo", ...)` para reemplazar el proceso actual por uno con `sudo`, manteniendo los argumentos originales.

```python
# Si no hay permisos, este llamado reiniciará el proceso
request_admin()
```

---

### `main_c() -> None`

Punto de entrada principal del módulo. Orquesta la verificación y solicitud de permisos.

**Flujo:**

1. Llama a `is_admin()`.
2. Si no hay permisos, llama a `request_admin()` y retorna (en Windows el proceso muere aquí; en Linux es reemplazado por `execvp`).
3. Si hay permisos, limpia la pantalla y muestra confirmación.

```python
from Modules.check_admin import main_c
main_c()
```

---

## Cómo funciona

```
Inicio de main_c()
       │
       ▼
  ¿is_admin()?
       │
   NO ─┤
       │
       ▼
  request_admin()
  ┌─────────────────────────────────────┐
  │ Windows: ShellExecuteW "runas"      │
  │ → Abre diálogo UAC                  │
  │ → sys.exit() del proceso original   │
  │                                     │
  │ Linux: os.execvp("sudo", ...)       │
  │ → Reemplaza el proceso actual       │
  └─────────────────────────────────────┘
       │
   SÍ ─┤
       │
       ▼
  Limpiar pantalla
  Imprimir confirmación
  input("Press Enter...")
```

---

## Ejemplos de Uso

### Uso básico en `MainGame.py`

```python
from Modules.check_admin import main_c

# Verifica y solicita permisos al inicio del juego
main_c()
```

### Verificar permisos sin solicitar elevación

```python
from Modules.check_admin import is_admin

if is_admin():
    print("Tenemos privilegios de administrador/root")
else:
    print("Ejecutando sin privilegios elevados")
```

### Solicitar elevación manualmente

```python
from Modules.check_admin import is_admin, request_admin

if not is_admin():
    print("Se requieren permisos de administrador para esta operación.")
    request_admin()
    # En Linux el código de aquí abajo no se ejecutará
    # En Windows tampoco (sys.exit fue llamado)
```

> **Nota:** En `MainGame.py`, la llamada `main_c()` está activa por defecto. Si no deseas que el juego requiera permisos de administrador, puedes comentarla o condicionarla con `CONFIG["ASK_FOR_ADMIN"]`.

---

## Notas y Referencias

- **UAC (User Account Control) en Windows:** El verbo `"runas"` en `ShellExecuteW` activa el diálogo de UAC. Documentación: [https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecutew](https://learn.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecutew)
- **`os.geteuid()`:** Retorna el UID efectivo del proceso. Solo disponible en Unix. Documentación: [https://docs.python.org/3/library/os.html#os.geteuid](https://docs.python.org/3/library/os.html#os.geteuid)
- **`os.execvp()`:** Reemplaza el proceso actual con un nuevo ejecutable. Es una llamada de la familia `exec`. Documentación: [https://docs.python.org/3/library/os.html#os.execvp](https://docs.python.org/3/library/os.html#os.execvp)
- **`ctypes` en Python:** Permite llamar a funciones de DLLs de Windows. Documentación: [https://docs.python.org/3/library/ctypes.html](https://docs.python.org/3/library/ctypes.html)
- **`pathlib.Path`:** Manejo moderno de rutas en Python. Documentación: [https://docs.python.org/3/library/pathlib.html](https://docs.python.org/3/library/pathlib.html)
