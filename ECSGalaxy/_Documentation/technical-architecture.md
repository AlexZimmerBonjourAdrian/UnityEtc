# Arquitectura Técnica

## Tabla de Contenidos

1. [Visión General](#visión-general)
2. [Arquitectura ECS](#arquitectura-ecs)
3. [Estructura de Directorios](#estructura-de-directorios)
4. [Sistemas Principales](#sistemas-principales)
5. [Flujo de Ejecución](#flujo-de-ejecución)
6. [Optimizaciones](#optimizaciones)
7. [Patrones de Diseño](#patrones-de-diseño)

---

## Visión General

Este proyecto implementa un juego RTS 4X (eXplore, eXpand, eXploit, eXterminate) utilizando **Unity ECS (Entity Component System)** y **DOTS**. La arquitectura está diseñada para:

- **Alto Rendimiento**: Manejar miles de entidades simultáneamente
- **Escalabilidad**: Paralelización masiva con Jobs
- **Modularidad**: Sistemas independientes y reutilizables
- **Mantenibilidad**: Código organizado y bien estructurado

---

## Arquitectura ECS

### Principios Fundamentales

El proyecto sigue el paradigma **Entity Component System**:

- **Entities**: Identificadores únicos para objetos del juego
- **Components**: Datos estructurados (IComponentData)
- **Systems**: Lógica que opera sobre componentes
- **Jobs**: Trabajos paralelos para procesamiento masivo

### Tipos de Componentes

#### Componentes de Datos (IComponentData)
```csharp
public struct Ship : IComponentData
{
    public float3 Velocity;
    public Entity NavigationTargetEntity;
    // ...
}
```

#### Componentes de Buffer (IBufferElementData)
```csharp
public struct FighterAction : IBufferElementData
{
    public Entity Entity;
    public float3 Position;
    public float Importance;
    // ...
}
```

#### Blob Assets
Datos inmutables compartidos para reducir memoria y mejorar rendimiento:
- `ShipData`
- `FighterData`
- `WorkerData`
- `TraderData`
- `BuildingData`

### Sistemas

Los sistemas implementan `ISystem` y pueden ser:
- **Partial Systems**: Permiten queries generadas automáticamente
- **Burst Compiled**: Compilados con Burst para máximo rendimiento
- **Parallel Jobs**: Ejecutados en paralelo cuando es posible

---

## Estructura de Directorios

```
Assets/Scripts/
├── Components/              # Componentes ECS
│   ├── Ship.cs             # Componentes de naves
│   ├── Planet.cs           # Componentes de planetas
│   ├── Building.cs         # Componentes de edificios
│   ├── Team.cs             # Sistema de equipos
│   ├── Config.cs           # Configuración global
│   └── ...
│
├── Authoring/              # Autoría en el editor
│   ├── ShipAuthoring.cs    # Convertir GameObjects a ECS
│   ├── PlanetAuthoring.cs
│   ├── BuildingAuthoring.cs
│   └── BlobAuthoring/      # Creación de Blob Assets
│
├── UI/                     # Sistema de interfaz
│   ├── UISystem.cs         # Sistema ECS para UI
│   ├── UIManager.cs        # Gestor de pantallas
│   ├── UIEvents.cs         # Eventos de UI
│   └── Screens/            # Pantallas individuales
│
└── Utilities/              # Utilidades y helpers
    ├── GameUtilities.cs    # Funciones auxiliares
    ├── MathUtilities.cs    # Utilidades matemáticas
    ├── AIProcessor.cs      # Procesamiento de IA
    ├── SpatialDatabase/   # Base de datos espacial
    └── ...
```

---

## Sistemas Principales

### Sistema de Inicialización

**GameInitializeSystem**
- **Ubicación**: `UpdateInGroup(BeginSimulationMainThreadGroup)`
- **Responsabilidad**: Inicializa el mundo del juego
- **Tareas**:
  1. Spawn de equipos y planetas home
  2. Spawn de planetas neutrales
  3. Spawn de lunas
  4. Spawn de naves iniciales
  5. Spawn de edificios iniciales
  6. Construcción de estructuras espaciales
  7. Cálculo de red de planetas
  8. Colocación de cámara

### Sistema de IA

**TeamAISystem**
- **Ubicación**: `UpdateAfter(BeginSimulationMainThreadGroup)`
- **Responsabilidad**: Evalúa estrategias y genera acciones
- **Proceso**:
  1. **Evaluación de Flotas**: Cuenta naves por tipo y equipo
  2. **Análisis de Planetas**: Calcula amenaza, seguridad, recursos
  3. **Generación de Acciones**:
     - `FighterAction`: Planetas a atacar/defender
     - `WorkerAction`: Planetas a capturar o edificios a construir
     - `TraderAction`: Planetas para intercambio de recursos
     - `FactoryAction`: Tipos de naves a producir
  4. **Asignación de Importancia**: Sistema de utilidades

**ApplyTeamSystem**
- **Responsabilidad**: Aplica decisiones de IA a las entidades

### Sistema de Naves

**ShipSystem** (1044 líneas)
- **Ubicación**: Después de `TeamAISystem` y `ApplyTeamSystem`
- **Jobs Principales**:
  - `ShipInitializeJob`: Inicialización de naves
  - `FighterInitializeJob`: Inicialización específica de fighters
  - `ShipNavigationJob`: Navegación y evasión de planetas
  - `FighterAIJob`: IA de combate y detección de objetivos
  - `WorkerAIJob`: IA de captura y construcción
  - `TraderAIJob`: IA de comercio
  - `FighterExecuteAttackJob`: Ejecución de ataques
  - `WorkerExecutePlanetCaptureJob`: Captura de planetas
  - `WorkerExecuteBuildJob`: Construcción de edificios
  - `TraderExecuteTradeJob`: Intercambio de recursos

### Sistema de Planetas

**PlanetSystem**
- **Ubicación**: Después de `BuildSpatialDatabaseGroup`
- **Jobs**:
  - `PlanetShipsAssessmentJob`: Evalúa naves alrededor de cada planeta
  - `PlanetConversionJob`: Gestiona conversión/captura de planetas
  - `PlanetResourcesJob`: Actualiza generación de recursos
  - `PlanetClearBuildingsDataJob`: Limpia datos de edificios

### Sistema de Edificios

**BuildingSystem** (551 líneas)
- **Jobs**:
  - `TurretInitializeJob`: Inicialización de torretas
  - `BuildingInitializeJob`: Inicialización de edificios
  - `BuildingConstructionJob`: Construcción de edificios
  - `TurretUpdateAttackJob`: Actualización de ataques de torretas
  - `TurretExecuteAttack`: Ejecución de ataques
  - `ResearchApplyToPlanetJob`: Aplicación de bonificaciones
  - `FactoryJob`: Producción de naves

### Sistema de Base de Datos Espacial

**BuildSpatialDatabasesSystem**
- **Responsabilidad**: Construye estructura espacial para queries rápidas
- **Optimización**: Paralelizable para máximo rendimiento
- **Uso**: Detección de objetivos, colisiones, queries de proximidad

### Sistema de Muerte

**DeathSystem**
- **Ubicación**: `OrderLast` en `SimulationSystemGroup`
- **Jobs**:
  - `BuildingDeathJob`: Maneja muerte de edificios
  - `ShipDeathJob`: Maneja muerte de naves
  - `FinalizedDeathJob`: Destruye entidades muertas

### Sistema VFX

**VFXSystem**
- **Responsabilidad**: Gestión eficiente de efectos visuales
- **Arquitectura**: Un VFXGraph por tipo, sin GameObjects por instancia
- **Tipos**: Explosiones, chispas de láser, propulsores

---

## Flujo de Ejecución

### Orden de Ejecución de Sistemas

```
InitializationSystemGroup
  └─ SimulationRateSystem (OrderFirst)
      └─ BeginSimulationMainThreadGroup
          ├─ GameInitializeSystem
          ├─ SimulationRateSystem (OrderFirst)
          └─ BuildSpatialDatabaseGroup
              ├─ ClearSpatialDatabaseSystem (OrderFirst)
              └─ BuildSpatialDatabasesSystem

  └─ SimulationSystemGroup
      ├─ TeamAISystem
      ├─ ApplyTeamSystem
      ├─ BuildSpatialDatabaseGroup
      │   └─ BuildSpatialDatabasesSystem
      ├─ PlanetSystem
      ├─ BuildingSystem
      ├─ ShipSystem
      ├─ LaserSystem
      ├─ TransformSystemGroup
      │   └─ CopyEntityLocalTransformAsLtWSystem (OrderLast)
      ├─ ShipPostTransformsSystem
      │   └─ VFXSystem
      ├─ DeathSystem (OrderLast)
      ├─ FinishInitializeSystem (OrderLast)
      └─ GameCameraSystem
          └─ EndSimulationEntityCommandBufferSystem

  └─ PresentationSystemGroup
      └─ MainCameraSystem
```

### Ciclo de Vida de una Entidad

1. **Inicialización**: `Initialize` component habilitado
2. **Update**: Sistemas procesan la entidad cada frame
3. **Muerte**: `Health.IsDead` se vuelve true
4. **Cleanup**: `DeathSystem` destruye la entidad

---

## Optimizaciones

### 1. Paralelización

- **Jobs Paralelos**: La mayoría de sistemas usan `ScheduleParallel`
- **Burst Compile**: Todos los sistemas críticos están compilados con Burst
- **SIMD**: Operaciones vectoriales aprovechan SIMD

### 2. Estructuras Espaciales

#### Spatial Database
- Grid uniforme para queries rápidas
- Queries por AABB optimizadas
- Filtrado por equipo y tipo sin lookups

#### Planet Navigation Grid
- Grid para evasión eficiente de planetas
- Acceso O(1) a planeta más cercano

#### Planets Network
- Red de planetas conectados
- Búsqueda de vecinos optimizada

### 3. Blob Assets

- Datos compartidos e inmutables
- Reduce memoria por entidad
- Mejora cache locality

### 4. Entity Command Buffers

- Operaciones diferidas para evitar race conditions
- ParallelWriter para escritura paralela segura

### 5. Component Enable/Disable

- Sistema de inicialización eficiente
- Múltiples componentes pueden inicializarse en el mismo frame

---

## Patrones de Diseño

### 1. Component-System Pattern
Arquitectura fundamental ECS donde componentes son datos y sistemas son lógica.

### 2. Job System Pattern
Paralelización masiva utilizando Jobs de Unity.

### 3. Spatial Partitioning
División del espacio para optimizar queries espaciales.

### 4. Utility AI
Sistema de IA basado en utilidades donde acciones se evalúan por importancia.

### 5. Strategy Pattern
Diferentes tipos de naves con comportamientos específicos (Fighter, Worker, Trader).

### 6. Observer Pattern
Sistema de eventos para UI (`UIEvents`, `EventRegistry`).

### 7. Singleton Pattern
Componentes singleton para configuración global (`Config`, `SpatialDatabaseSingleton`).

---

## Consideraciones de Rendimiento

### Mejores Prácticas Implementadas

1. **Queries Eficientes**: Uso de `EntityQueryBuilder` con filtros específicos
2. **Chunk Iteration**: Procesamiento a nivel de chunk cuando es posible
3. **No Allocations**: Uso de `Allocator.Temp` y `Allocator.TempJob` apropiadamente
4. **Native Collections**: Uso de `NativeArray`, `NativeList` para datos temporales
5. **Lookup Caching**: Caché de ComponentLookup dentro de jobs
6. **Batch Operations**: Operaciones en batch cuando es posible

### Métricas de Rendimiento

El proyecto está diseñado para manejar:
- Miles de naves simultáneas
- Cientos de planetas
- Decenas de equipos
- 60+ FPS en hardware moderno

---

## Extensiones Futuras

### Áreas de Mejora Identificadas

1. **Refactorización de Sistemas Grandes**:
   - `ShipSystem` (1044 líneas) podría dividirse
   - `TeamAISystem` (1021 líneas) podría modularizarse más

2. **Documentación de Código**:
   - Comentarios XML en métodos públicos
   - Documentación de parámetros complejos

3. **Tests**:
   - Unit tests para utilidades
   - Integration tests para sistemas críticos

4. **Profiling Tools**:
   - Herramientas de profiling personalizadas
   - Métricas de rendimiento integradas

---

## Referencias

- [Unity ECS Documentation](https://docs.unity3d.com/Packages/com.unity.entities@latest)
- [DOTS Best Practices](https://docs.unity3d.com/Packages/com.unity.entities@latest/manual/index.html)
- [Burst Compiler](https://docs.unity3d.com/Packages/com.unity.burst@latest)

