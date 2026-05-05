# `EquipmentSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Gestor: `EquipmentManager`](#gestor-equipmentmanager)
3. [Slots de Equipamiento](#slots-de-equipamiento)
4. [Lógica de Aplicación de Stats](#lógica-de-aplicación-de-stats)
5. [Gestión de Múltiples Anillos](#gestión-de-múltiples-anillos)
6. [Interfaz: Menú de Equipamiento](#interfaz-menú-de-equipamiento)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`EquipmentSystem.py` es el núcleo que gestiona cómo los objetos afectan las estadísticas del jugador. Se encarga de "instalar" y "desinstalar" los atributos de los ítems de forma dinámica cuando el jugador decide equipar o desequipar una pieza de su arsenal.

---

## Gestor: `EquipmentManager`

Clase estática que contiene la lógica pura de manipulación de slots.

### Funciones Principales
- `apply_item_stats(player, item)`: Suma los bonus del ítem (Vida, Defensa, etc.) a los atributos base del jugador.
- `remove_item_stats(player, item)`: Resta los bonus, devolviendo al jugador a su estado original.
- `equip_item(...)`: Maneja el intercambio de piezas. Si ya hay algo en el slot, lo envía al inventario automáticamente.
- `unequip_item(...)`: Libera un slot específico.

---

## Slots de Equipamiento

El sistema reconoce los siguientes espacios:

- **Arma (`weapon`)**: Modifica el daño base y tipo de ataque.
- **Casco (`helmet`)**: Principalmente defensa.
- **Armadura (`armor`)**: Máxima protección y bonus de HP.
- **Botas (`boots`)**: Defensa y agilidad.
- **Anillos (`ring`, `ring2`, `ring3`)**: Tres slots para accesorios con efectos variados.
- **Herramienta (`tool`)**: Slot para picos, hachas, etc.

---

## Lógica de Aplicación de Stats

Cuando se equipa un objeto (ej. una Armadura con +20 HP y +5 Defensa Física):
1. El sistema llama a `get_item_stats()`.
2. Suma `20` a `player.Health_max`.
3. Suma `5` a `player.DEF[0]` (Físico).
4. El objeto se marca como `AppliedStats = True` para evitar duplicaciones por errores de código.

Al desequipar, se realiza el proceso inverso.

---

## Gestión de Múltiples Anillos

Dado que el jugador puede llevar hasta **3 anillos**, el sistema incluye una lógica especial:
- Al equipar un anillo, si hay un slot libre, lo ocupa automáticamente.
- Si los 3 están llenos, abre un submenú preguntando cuál de los 3 anillos actuales se desea reemplazar.

---

## Interfaz: Menú de Equipamiento

### `equip_menu(player, inventory)`
Menú interactivo que permite:
1. **Equipar**: Filtra el inventario mostrando solo ítems compatibles.
2. **Desequipar**: Permite elegir qué pieza quitarse.
3. **Ver Estado**: Muestra un desglose visual de cada pieza equipada y sus bonus actuales.

---

## Ejemplos de Uso

### Equipar un arma desde el código

```python
from Modules.ModulesFunctions.EquipmentSystem import EquipmentManager

# espada es una instancia de cWeapon
success = EquipmentManager.equip_item(player, espada, inventory)
if success:
    print(f"Ahora empuñas {espada.Name}")
```

### Consultar stats totales (Base + Equipo)

```python
stats = EquipmentManager.get_total_stats(player)
print(f"HP Máximo actual: {stats['Health_max']}")
```
