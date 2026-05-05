# `ItemManager.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Clase `ItemManager`](#clase-itemmanager)
3. [Estructura del JSON (`DataItems.json`)](#estructura-del-json-dataitemsjson)
4. [Indexación y Búsqueda](#indexación-y-búsqueda)
5. [Factoría de Objetos](#factoría-de-objetos)
6. [Gestión de CRUD](#gestión-de-crud)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`ItemManager.py` es el pilar central que gestiona la base de datos de todos los objetos del juego. Actúa como un motor de persistencia y una factoría que transforma datos planos de JSON en objetos complejos de Python con los que el motor de juego puede interactuar.

---

## Clase `ItemManager`

Maneja la carga, indexación y guardado de ítems. Implementa el patrón **Singleton** a través de la función `get_item_manager()`.

### Atributos clave
- `items_data`: El diccionario crudo cargado desde el archivo.
- `items_by_id`: Índice para acceso rápido por ID (soporta prefijos para materiales).
- `items_by_name`: Índice para acceso por nombre (case-insensitive).

---

## Estructura del JSON (`DataItems.json`)

Los ítems se organizan en categorías prefijadas con `TYPE:`:
- `TYPE:Weapons`: Armas.
- `TYPE:Armor`, `TYPE:Helmet`, etc: Equipamiento.
- `TYPE:Consumable`: Pociones y objetos de un solo uso.
- `TYPE:Tools`: Picos, hachas, etc.
- `TYPE:Materials`: Recursos para crafteo y construcción.

---

## Indexación y Búsqueda

Debido a que el juego tiene cientos de ítems, el gestor construye mapas de búsqueda al iniciar:
- **IDs Estáticos**: Se usan para equipo y armas.
- **IDs de Materiales**: Se indexan internamente como `mat_{ID}` para evitar colisiones con los IDs estáticos de otras categorías.

---

## Factoría de Objetos

El método `create_item_object(...)` es el corazón del sistema. Dependiendo del campo `Type` en el JSON, instancia la clase correspondiente:
- `Weapon` -> `cWeapon`
- `Armor`/`Helmet`... -> `cEquippableItems`
- `Consumable` -> `cObject`
- `Tool` -> `cTool`
- `Material` -> `cMaterial`

---

## Gestión de CRUD

El `ItemManager` no es solo de lectura. Permite la manipulación dinámica de la base de datos:
- `add_item()`: Inserta un nuevo ítem y actualiza el JSON.
- `modify_item()`: Cambia atributos de un ítem existente (ej. ajustar daño o precio).
- `delete_item()`: Elimina una entrada permanentemente.

---

## Ejemplos de Uso

### Obtener el gestor y cargar un arma

```python
from Modules.ModulesManager.ItemManager import get_item_manager

im = get_item_manager()
# Cargar espada con ID 5
espada = im.create_item_object(5)
```

### Listar ítems respetando el descubrimiento del jugador

```python
# Muestra nombres reales solo si el jugador ya los encontró,
# si no, aparecerán censurados con ████
im.list_all_items(detailed=True, discovered_ids=player.discovered_items)
```

### Añadir un ítem nuevo mediante código

```python
nuevo_item = {
    "STATIC_ID": 999,
    "Name": "Super Espada",
    "Type": "Weapon",
    "DMG": [100, 0, 0, 0, 0],
    "Gold_Cost": 5000
}
im.add_item("Weapons", "super_espada", nuevo_item)
```
---

## Notas Técnicas

- El sistema soporta arrays de daño y defensa de 5 elementos (Físico, Térmico, Tierra, Eléctrico, Profundo).
- La carga es automática al llamar por primera vez a `get_item_manager()`.
