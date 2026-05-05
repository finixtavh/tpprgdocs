# `GatheringMasterySystem.py`

## Índice

1. [Descripción General](#descripción-general)
2. [Habilidades de Recolección (Skills)](#habilidades-de-recolección-skills)
3. [Clases de Objetos](#clases-de-objetos)
    - [`cMaterial`](#cmaterial)
    - [`cTool`](#ctool)
4. [El Minijuego de Recolección](#el-minijuego-de-recolección)
5. [Cálculo de Recompensas](#cálculo-de-recompensas)
6. [Integración con `cFarm`](#integración-con-cfarm)
7. [Ejemplos de Uso](#ejemplos-de-uso)

---

## Descripción General

`GatheringMasterySystem.py` gestiona todas las actividades de recolección de recursos en el mundo. Implementa un sistema de progresión de niveles para habilidades no combativas y un minijuego interactivo de timing que determina la velocidad y éxito de la recolección.

---

## Habilidades de Recolección (Skills)

El sistema soporta las siguientes disciplinas:
- **Minería (Mining)**: Extracción de menas y piedras.
- **Tala (Woodcutting)**: Obtención de diversos tipos de madera.
- **Pesca (Fishing)**: Captura de peces en zonas acuáticas.
- **Herboristería (Herbalism)**: Recolección de plantas y hongos.
- **Peletería (Skinning)**: Obtención de cueros de animales.
- **Astrología (Astrology)**: Recolección de materiales celestiales.

### Bonus por Nivel
Cada nivel ganado en una skill otorga:
- **+5% Eficiencia**: Reduce el tiempo necesario para recolectar.
- **+10% XP Factor**: Aumenta la experiencia ganada en esa skill.

---

## Clases de Objetos

### `cMaterial`
Define los recursos naturales.
- `Hardness`: Determina cuánto tiempo toma recolectarlo.
- `Level_Req`: Nivel mínimo de skill necesario (basado en la rareza).
- `SkillName`: La disciplina requerida para este material.

### `cTool`
Define las herramientas (picos, hachas, etc.).
- `Efficiency`: Multiplicador que reduce el tiempo de recolección.
- `ToolTypeStr`: Define qué materiales puede procesar (ej. "TYPE_M:wood").

---

## El Minijuego de Recolección

Cuando el jugador interactúa con un recurso, se inicia un minijuego visual:
1. **Barra de Progreso**: Muestra cuánto falta para terminar.
2. **Zona de Acierto (Verde)**: Una zona que se mueve aleatoriamente.
3. **Marcador (▼)**: Oscila de izquierda a derecha.

**Mecánica**: Pulsar **Espacio** cuando el marcador esté en la zona verde reduce el tiempo restante en un **15% (acumulable)**.

---

## Cálculo de Recompensas

Tras completar el tiempo de recolección:
1. **Experiencia**: Se otorga EXP a la skill correspondiente basada en la `Hardness` del material.
2. **Cantidad**: Se realiza un roll de probabilidad:
    - **1x**: 80% base.
    - **2x**: 15% base.
    - **3x**: 5% base.
    - El "Poder" (Herramienta + Skill) aumenta significativamente las probabilidades de obtener 2x y 3x.

---

## Integración con `cFarm`

La clase `cFarm` actúa como el gestor global (`Gathering_init`).
- Mantiene el estado de todos los niveles de skill del jugador.
- Maneja la lógica de renderizado del minijuego usando `Rich.Live`.

---

## Ejemplos de Uso

### Iniciar una recolección

```python
from Modules.ModulesFunctions.GatheringMasterySystem import Gathering_init

try:
    # material y tool son instancias de cMaterial y cTool
    cantidad = Gathering_init.gather(material, tool)
    inventory.addObject(material, cantidad)
except PermissionError as e:
    print(f"No puedes recolectar esto: {e}")
```

### Consultar nivel de una skill

```python
lvl = Gathering_init.gathering["mining"].current_level
print(f"Tu nivel de minería es {lvl}")
```
