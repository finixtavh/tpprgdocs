# `ItemModifierSystem.py`

## Índice
1. [Descripción General](#descripción-general)
2. [Dependencias e Inyecciones](#dependencias)
3. [Constantes y Variables Globales](#constantes)
4. [Clases y Estructuras de Datos](#clases)
5. [Funciones del Módulo (API)](#funciones)

---

## 1. Descripción General
El `ItemModifierSystem` (IMS) es el motor matemático encargado de generar y aplicar afijos y sufijos aleatorios procedurales a las armas, equipables y herramientas (ya sea por crafteo, drops de combates o re-rolleos en yunques). 
Aplica modificadores matemáticos (multiplicadores de daño, defensa o recolección) basados en una tirada de suerte (roll) fuertemente dependiente de la configuración en `config.json`.

---

## 2. Dependencias e Inyecciones
- **Inyecciones**:
  - `cWeapon`, `cEquippableItems` de `setup.py` (para validación de tipos mediante `isinstance`).
  - `cTool` de `GatheringMasterySystem` (para validación de herramientas).
  - Constantes de probabilidad (`GLOBAL_CRAFTING_QUALITY_MODIFIER`) leídas desde la inicialización global en `config.json` a través de `MainGame.py` o los sistemas de crafteo.

---

## 3. Constantes y Variables Globales
- **Librería de Modificadores (Diccionarios Estáticos)**:
  - `WEAPON_MODIFIERS`: Diccionario de diccionarios. Clasificado en `bad`, `normal`, `good`. Contiene mutadores para el daño (`dmg_mult`) y oro (`cost_mult`).
  - `ARMOR_MODIFIERS`: Mutadores para la defensa (`def_mult`).
  - `TOOL_MODIFIERS`: Mutadores para recolección (`speed_mod`, `yield_mod`, `durability_mod`).

---

## 4. Clases y Estructuras de Datos

*(Nota: Este módulo no implementa clases de instanciado continuo, opera directamente manipulando los objetos que recibe por referencia en sus funciones API).*

---

## 5. Funciones del Módulo (API)

- `get_random_modifier(item_type: str, roll: float) -> Tuple[str, Dict]`
  - **Propósito**: Consulta la librería estática correspondiente según el tipo (`weapon`, `armor`, `tool`) y el resultado de la tirada (`roll`), retornando un modificador aleatorio de esa categoría de rareza.
  - **Parámetros**: 
    - `item_type`: String (ej. `"weapon"`).
    - `roll`: Flotante del 0.0 al 1.0 (o más, si hay bonos).
  - **Retornos**: Una tupla con `(Nombre del Afijo, Diccionario de Mutadores)`.

- `apply_modifier_to_item(item, mod_name: str, mod_effects: Dict)`
  - **Propósito**: Aplica las matemáticas del modificador a los atributos base del objeto, mutándolo in-place.
  - **Parámetros**: 
    - `item`: Instancia de `cWeapon`, `cEquippableItems`, o `cTool`.
    - `mod_name`: String con el prefijo a agregar al nombre original (ej: `"Legendario"`).
    - `mod_effects`: Diccionario con las llaves numéricas (ej: `{"dmg_mult": 1.5}`).
  - **Comportamiento Interno**:
    - Si es `cWeapon`: Multiplica todos los elementos mayores a 0 del array `DMG` e incrementa el `Gold_Cost`.
    - Si es `cEquippableItems`: Multiplica el array `Defense`.
    - Si es `cTool`: Suma/Resta enteros en `Speed`, `Yield` y `Durability`.
    - Guarda en `item.modifier_applied` el `mod_name` para que el sistema de Reforjado pueda remover el texto del nombre posteriormente.

- `remove_current_modifier(item)`
  - **Propósito**: Revierte el objeto a su estado base original antes de aplicar un nuevo *roll* en la mecánica de Reforjado.
  - **Comportamiento**: Lee `item.base_stats` (guardado por `apply_modifier_to_item`), restaura arrays y costos, y elimina el prefijo del nombre usando `.replace(f"{item.modifier_applied} ", "")`.

- `roll_and_apply_modifier(item, roll_bonus: float = 0.0)`
  - **Propósito**: Función envoltorio (wrapper) principal que debe ser llamada por los sistemas externos (Crafteo o Loot).
  - **Parámetros**: 
    - `item`: Objeto recién creado.
    - `roll_bonus`: Flotante base de suerte provisto por la zona o el skill tree.
  - **Comportamiento**:
    1. Lanza un `random.random()`.
    2. Le suma `roll_bonus`.
    3. Interpreta el tipo de objeto usando `isinstance`.
    4. Llama a `get_random_modifier` y luego a `apply_modifier_to_item`.
