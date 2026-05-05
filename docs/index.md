# TPPRPG Engine - Documentación Técnica v0.9

Bienvenido a la documentación técnica oficial de **TPPRPG**, un motor de RPG modular desarrollado en Python y optimizado para interfaces de terminal modernas mediante la librería `Rich`.

---

## Introducción

TPPRPG no es solo un juego, sino un ecosistema extensible que permite la creación de experiencias de rol complejas. El motor gestiona desde el combate táctico multi-elemental hasta la simulación geopolítica de gremios y un sistema avanzado de mods.

---

## Arquitectura del Motor

El siguiente diagrama muestra cómo interactúan los diferentes módulos del motor:

```mermaid
graph TD
    Main[MainGame.py] --> Input[input_utils.py]
    Main --> Managers[Managers]
    Main --> UI[Menus]
    
    subgraph Managers
        IM[ItemManager]
        BM[BuffManager]
        SM[ShopManager]
        ZM[ZoneManager]
    end
    
    subgraph Core Systems
        Combat[Combat Loop]
        Guild[Guild System]
        Base[Base System]
        Mod[Mod Loader]
    end
    
    Managers --> Combat
    Combat --> Player[cPlayer Class]
    Combat --> Enemy[cEnemy Class]
    Mod --> Managers
    Mod --> Core Systems
```

---

## Características Principales

- **Combate Multi-Tipo**: Sistema de daño y defensa basado en 5 elementos (Físico, Térmico, Tierra, Eléctrico, Profundo).
- **Simulación de Gremios**: Una IA estratégica que gestiona territorios, diplomacia y recursos al estilo "Grand Strategy".
- **Sistema de Mods**: Inyección dinámica de contenido vía JSON y Python sin modificar el núcleo.
- **Gestión de Base**: Construcción de edificios con efectos pasivos y minijuegos de entrenamiento.
- **Persistencia Robusta**: Sistema de guardado en slots con soporte para estados complejos y descubrimiento de ítems.

---

## Enlaces del Proyecto

- **Repositorio del Proyecto**: [GitHub TPPRPG](https://github.com/finixtavh/TPPRPG)
- **Documentación Original**: [TPPRPG Docs](https://github.com/finixtavh/tpprgdocs)

---

## Autor

Desarrollado con ❤️ por **finixtavh (aka lole)**.
