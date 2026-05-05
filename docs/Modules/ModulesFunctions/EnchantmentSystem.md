# `EnchantmentSystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `cEnchantment`](#clase-cenchantment)
3. [Lógica de Familias y Niveles](#lógica-de-familias-y-niveles)
4. [Efectos por Tipo de Objeto](#efectos-por-tipo-de-objeto)
5. [Carga y Registro](#carga-y-registro)
6. [Interfaz: Mesa de Encantamientos](#interfaz-mesa-de-encantamientos)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`EnchantmentSystem.py` permite mejorar las estadísticas de armas, armaduras y herramientas mediante "encantamientos". Este sistema es dinámico y permite que un mismo encantamiento suba de nivel o incluso cambie su nombre y efectos al alcanzar su máximo potencial.

---

## Clase `cEnchantment`

Define las propiedades de un encantamiento cargado desde `DataEnchantments.json`.

- `ENCHANT_ID`: Identificador único.
- `TargetClass`: Clase de objeto compatible (`cWeapon`, `cEquippableItems`, `cTool`).
- `Effects`: Diccionario de modificadores (ej. `{"dmg_bonus": 1.2}`).
- `MaxLevel`: Nivel máximo alcanzable (típicamente 5).
- `max_level_name`: Nombre especial que adopta el encantamiento al llegar al máximo (ej. "Filo V" -> "Excalibur").

---

## Lógica de Familias y Niveles

Para evitar que un ítem se llene de múltiples versiones del mismo efecto, el sistema utiliza `family_id`.
- Si aplicas un encantamiento de una familia que el ítem ya posee, **subirá de nivel** (I → II → III...) en lugar de ocupar un nuevo slot.
- Un ítem tiene un máximo de **5 slots** para familias únicas.

---

## Efectos por Tipo de Objeto

### Armas (`cWeapon`)
- `dmg_bonus`: Daño porcentual o fijo por tipo.
- `crit_chance` / `crit_damage`: Mejora de críticos.
- `lifesteal`: Robo de vida basado en el daño infligido.
- `special_effect`: Efectos únicos como "Sangrado" o "Quemadura".

### Equipamiento (`cEquippableItems`)
- `defense_bonus`: Mejora de resistencias.
- `health_bonus`: Puntos de vida extra.
- `reflect_damage`: Devuelve un porcentaje del daño al atacante.
- `thorns`: Daño fijo devuelto al ser golpeado.

### Herramientas (`cTool`)
- `efficiency_bonus`: Multiplicador de eficiencia en recolección.
- `fortune`: Probabilidad de obtener ítems extra.
- `auto_smelt`: Funde minerales automáticamente al picar.

---

## Carga y Registro

El catálogo global `ENCHANTMENTS_CATALOG` se llena automáticamente al importar el módulo leyendo `Data/DataEnchantments.json`.

---

## Interfaz: Mesa de Encantamientos

### `open_enchantment_menu(player, inventory)`
Permite al jugador interactuar con el sistema:
1. **Selección de Objeto**: Lista todo el equipo equipado y en el inventario que sea encantable.
2. **Catálogo Místico**: Muestra solo los encantamientos compatibles con el objeto seleccionado y sus costes en oro.
3. **Infundir**: Aplica el nuevo encantamiento o mejora el existente.

---

## Ejemplos de Uso

### Aplicar efectos de encantamiento en combate

```python
from Modules.ModulesFunctions.EnchantmentSystem import get_item_enchants

# Obtener los encantamientos de un arma
enchants = get_item_enchants(player.weapon)
for e_data in enchants:
    # Lógica para aplicar los bonus...
    pass
```

### Consultar nombre visual

```python
# Retornará algo como "Filo III" o "Poder Sagrado"
nombre_bonito = enchant_obj.get_display_name()
```
