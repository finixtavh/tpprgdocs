# `__init__.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Propósito del archivo](#propósito-del-archivo)
3. [Cómo funciona](#cómo-funciona)
4. [Notas y Referencias](#notas-y-referencias)

---

## Descripción General

El archivo `__init__.py` convierte el directorio `Modules/` en un **paquete de Python**. Su contenido actual es mínimo (un comentario), pero su sola presencia es suficiente para que Python reconozca la carpeta como un paquete importable.

---

## Propósito del archivo

Este archivo no define variables, clases ni funciones. Su única función es estructural: al existir dentro de un directorio, le indica a Python que ese directorio debe tratarse como un paquete.

Esto permite que desde otros archivos del proyecto se hagan imports como:

```python
from Modules.setup import cPlayer
from Modules.Loader import Read, Write
from Modules.BuffManager import get_buff_manager
```

Sin este archivo, Python no sabría que `Modules/` es un paquete y los imports fallarían con un `ModuleNotFoundError`.

---

## Cómo funciona

Python busca `__init__.py` al resolver imports de paquetes. Cuando encuentra este archivo en un directorio, ejecuta su contenido (en este caso vacío) y registra el directorio como paquete. En versiones modernas de Python (3.3+) existen los *namespace packages* que no requieren `__init__.py`, pero usarlo explícitamente sigue siendo la práctica estándar y más compatible.

### Estructura del paquete

```
Modules/
│
├── __init__.py            ← Este archivo
├── setup.py
├── Loader.py
├── BuffManager.py
├── EnchantmentSystem.py
├── FarmingSystem.py
├── GatheringMasterySystem.py
├── ShopManager.py
├── ItemManager.py
├── EquipmentSystem.py
├── ModLoader.py
├── check_admin.py
└── MainGame.py
```

---

## Notas y Referencias

- **Referencia oficial de Python sobre paquetes:** [https://docs.python.org/3/reference/import.html#package-relative-imports](https://docs.python.org/3/reference/import.html#package-relative-imports)
- **PEP 328 – Imports relativos:** [https://peps.python.org/pep-0328/](https://peps.python.org/pep-0328/)
- En el futuro, este archivo puede usarse para exponer una API pública del paquete, por ejemplo definiendo `__all__` para controlar qué se exporta al hacer `from Modules import *`.
