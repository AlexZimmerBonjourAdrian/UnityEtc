# ECS Galaxy Sample

<img src="./_Documentation/Images/GalaxySample.gif" alt="game" height="350"/>

> See [ProjectVersion](./ProjectSettings/ProjectVersion.txt) for minimum supported Unity version.

## Descripción

Este proyecto es una simulación a gran escala de equipos de naves espaciales luchando por el control de planetas, implementada utilizando **Unity ECS (Entity Component System)** y **DOTS** (Data-Oriented Technology Stack).

### Características Principales

- 🚀 **Sistema ECS completamente basado en DOTS** para máximo rendimiento
- ⚔️ **Sistema de combate multi-equipo** con IA estratégica
- 🏗️ **Sistema de construcción** de edificios y naves
- 🌍 **Sistema de planetas y recursos** con generación procedural
- 🤖 **IA de utilidades** para toma de decisiones estratégicas
- 🔄 **Sistemas espaciales optimizados** para queries rápidas
- 💥 **Sistema VFX eficiente** sin GameObjects por instancia
- 🔀 **Paralelización masiva** con Jobs y Burst Compile

### Tipos de Naves

- **Fighters**: Defienden planetas capturados y atacan planetas enemigos
- **Workers**: Capturan planetas y construyen edificios en lunas
- **Traders**: Distribuyen recursos uniformemente entre planetas capturados

### Tipos de Edificios

- **Factories**: Producen naves basándose en las necesidades estratégicas
- **Turrets**: Defienden planetas y edificios disparando a enemigos en rango
- **Research**: Proporcionan bonificaciones que afectan velocidades de construcción, generación de recursos y atributos de naves

## Inicio Rápido

Para ejecutar la simulación, abre la escena `Main` y presiona Play. Aparecerá un menú UI que te permitirá configurar los parámetros de la simulación. Una vez listo, presiona el botón "Simulate" y la simulación comenzará. Durante el juego, el menú puede ser abierto nuevamente para ajustar algunos parámetros en tiempo real.

### Controles

- **WASD + Mouse** - Control de cámara (solo cuando el menú está oculto)
- **Escape** - Alternar menú en juego
- **Z** - Alternar entre 3 modos de cámara (cámara libre, órbita planeta, órbita nave)
- **X** - Cambiar objetivo de cámara en los modos de órbita
- **Mouse Wheel** - Zoom in/out para cámaras de órbita
- **Left Shift** - Aumentar velocidad de cámara libre

### Requisitos

- Unity 2022.3 LTS o superior
- Packages DOTS requeridos (Entities, Collections, Mathematics, Transforms, Burst)

## Estructura del Proyecto

```
Assets/Scripts/
├── Components/          # Componentes ECS (Ship, Planet, Building, Team, etc.)
├── Authoring/          # MonoBehaviours para autoría en el editor
├── UI/                 # Sistema de interfaz de usuario
└── Utilities/          # Utilidades y helpers del proyecto
```

## Documentación

### Para Usuarios
- [Game Overview](./_Documentation/game-overview.md) - Descripción del juego y mecánicas

### Para Desarrolladores
- [Code Overview](./_Documentation/code-overview.md) - Visión general del código
- [Arquitectura Técnica](./_Documentation/technical-architecture.md) - Arquitectura detallada del proyecto
- [Guía de Desarrollo](./_Documentation/development-guide.md) - Guía para contribuir y extender el proyecto
- [Referencia de Sistemas](./_Documentation/systems-reference.md) - Documentación detallada de sistemas
- [Flujo de Sistemas](./_Documentation/systems-flow.md) - Diagrama de flujo de ejecución de sistemas
- [Debug Views](./_Documentation/debug-views.md) - Visualizaciones de debug disponibles

## Arquitectura

El proyecto utiliza una arquitectura **ECS (Entity Component System)** que permite:

- **Alto rendimiento**: Paralelización masiva con Jobs
- **Escalabilidad**: Maneja miles de entidades eficientemente
- **Modularidad**: Sistemas independientes y reutilizables
- **Optimización espacial**: Estructuras de datos especializadas para queries rápidas

### Flujo Principal de Sistemas

1. **Inicialización**: `GameInitializeSystem` configura el mundo
2. **IA de Equipos**: `TeamAISystem` evalúa estrategias y genera acciones
3. **Navegación**: `ShipSystem` maneja movimiento y combate
4. **Planetas**: `PlanetSystem` gestiona recursos y conversión
5. **Construcción**: `BuildingSystem` maneja edificios y producción
6. **Visualización**: Sistemas de cámara y VFX

Para más detalles, consulta [Arquitectura Técnica](./_Documentation/technical-architecture.md).

## Contribuciones

Este es un proyecto de muestra que demuestra las capacidades de Unity ECS. Las contribuciones son bienvenidas. Por favor, consulta la [Guía de Desarrollo](./_Documentation/development-guide.md) antes de contribuir.

## Licencia

Ver [LICENSE.md](./LICENSE.md) para más información.
