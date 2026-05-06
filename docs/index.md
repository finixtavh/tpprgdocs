# TPPRPG Engine - v0.9

Bienvenido a la documentación técnica oficial de **TPPRPG**, un motor de RPG modular desarrollado en Python y optimizado para interfaces de terminal modernas mediante la librería `Rich`.

---

## Introducción

{WIP}

---

## Arquitectura del Motor

El siguiente diagrama muestra cómo interactúan los diferentes módulos del motor:

``` mermaid
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

## Enlaces del Proyecto

- **Repositorio del Proyecto**: [GitHub TPPRPG](https://github.com/finixtavh/TPPRPG)
- **Documentación Original**: [TPPRPG Docs](https://github.com/finixtavh/tpprgdocs)

---

## Autor

Desarrollado por **finixtavh**.
