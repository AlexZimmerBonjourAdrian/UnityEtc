# Diagrama de Flujo de Sistemas

## Flujo Principal de Ejecución

```
┌─────────────────────────────────────────────────────────────────┐
│                    InitializationSystemGroup                    │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         SimulationRateSystem (OrderFirst)                 │  │
│  │         - Control de velocidad de simulación              │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           BeginSimulationMainThreadGroup                  │  │
│  │                                                           │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │        GameInitializeSystem                          │  │  │
│  │  │        - Spawn equipos y planetas                    │  │  │
│  │  │        - Crea estructuras espaciales                 │  │  │
│  │  │        - Inicializa cámara                           │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  │  ┌────────────────────────────────────────────────────┐  │  │
│  │  │        BuildSpatialDatabaseGroup                     │  │  │
│  │  │                                                      │  │  │
│  │  │  ┌──────────────────────────────────────────────┐  │  │  │
│  │  │  │  ClearSpatialDatabaseSystem (OrderFirst)      │  │  │  │
│  │  │  │  - Limpia base de datos espacial              │  │  │  │
│  │  │  └──────────────────────────────────────────────┘  │  │  │
│  │  │                                                      │  │  │
│  │  │  ┌──────────────────────────────────────────────┐  │  │  │
│  │  │  │  BuildSpatialDatabasesSystem                  │  │  │  │
│  │  │  │  - Reconstruye base de datos espacial         │  │  │  │
│  │  │  └──────────────────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    SimulationSystemGroup                        │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         SimulationRateSystem (OrderFirst)                │  │
│  │         - Actualiza DeltaTime                             │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         TeamAISystem                                      │  │
│  │         - Evalúa estrategias de equipos                  │  │
│  │         - Genera acciones para naves y edificios         │  │
│  │                                                           │  │
│  │         Proceso:                                           │  │
│  │         1. Evaluación de flotas                          │  │
│  │         2. Análisis de planetas                          │  │
│  │         3. Generación de acciones                        │  │
│  │         4. Asignación de importancia                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         ApplyTeamSystem                                   │  │
│  │         - Aplica decisiones de IA                        │  │
│  └──────────────────────────────────────────────────────────┘
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         BuildSpatialDatabaseGroup (reconstrucción)       │  │
│  │         - Limpia y reconstruye cada frame                 │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         PlanetSystem                                      │  │
│  │         - Evalúa naves alrededor de planetas              │  │
│  │         - Maneja conversión/captura                      │  │
│  │         - Genera recursos                                │  │
│  │                                                           │  │
│  │         Jobs:                                             │  │
│  │         ├─ PlanetShipsAssessmentJob                      │  │
│  │         ├─ PlanetConversionJob                           │  │
│  │         ├─ PlanetResourcesJob                            │  │
│  │         └─ PlanetClearBuildingsDataJob                  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         BuildingSystem                                    │  │
│  │         - Inicializa edificios                           │  │
│  │         - Maneja construcción                            │  │
│  │         - Actualiza torretas                             │  │
│  │         - Produce naves en fábricas                     │  │
│  │                                                           │  │
│  │         Jobs:                                             │  │
│  │         ├─ TurretInitializeJob                         │  │
│  │         ├─ BuildingInitializeJob                       │  │
│  │         ├─ BuildingConstructionJob                      │  │
│  │         ├─ TurretUpdateAttackJob                        │  │
│  │         ├─ TurretExecuteAttack                          │  │
│  │         ├─ ResearchApplyToPlanetJob                     │  │
│  │         └─ FactoryJob                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         ShipSystem                                        │  │
│  │         - Inicializa naves                              │  │
│  │         - Maneja navegación                             │  │
│  │         - Procesa IA de naves                           │  │
│  │         - Ejecuta combate                               │  │
│  │                                                           │  │
│  │         Jobs:                                             │  │
│  │         ├─ ShipInitializeJob                           │  │
│  │         ├─ FighterInitializeJob                        │  │
│  │         ├─ ShipNavigationJob (Parallel)                 │  │
│  │         ├─ FighterAIJob (Parallel)                      │  │
│  │         ├─ WorkerAIJob (Parallel)                      │  │
│  │         ├─ TraderAIJob (Parallel)                       │  │
│  │         ├─ FighterExecuteAttackJob                      │  │
│  │         ├─ WorkerExecutePlanetCaptureJob                │  │
│  │         ├─ WorkerExecuteBuildJob                        │  │
│  │         └─ TraderExecuteTradeJob                       │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         LaserSystem                                       │  │
│  │         - Actualiza proyectiles láser                   │  │
│  │         - Detecta colisiones                            │  │
│  │         - Aplica daño                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         TransformSystemGroup                              │  │
│  │         - Actualiza transformaciones                     │  │
│  │                                                           │  │
│  │         ┌─────────────────────────────────────────────┐  │  │
│  │         │  CopyEntityLocalTransformAsLtWSystem          │  │  │
│  │         │  (OrderLast)                                 │  │  │
│  │         │  - Copia transformaciones para LODs          │  │  │
│  │         └─────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         ShipPostTransformsSystem                        │  │
│  │                                                           │  │
│  │         ┌─────────────────────────────────────────────┐  │  │
│  │         │  VFXSystem                                   │  │  │
│  │         │  - Actualiza efectos visuales              │  │  │
│  │         │  - Procesa requests de VFX                 │  │  │
│  │         └─────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         DeathSystem (OrderLast)                            │  │
│  │         - Maneja muerte de entidades                     │  │
│  │         - Crea efectos de explosión                      │  │
│  │         - Destruye entidades muertas                     │  │
│  │                                                           │  │
│  │         Jobs:                                             │  │
│  │         ├─ BuildingDeathJob                             │  │
│  │         ├─ ShipDeathJob                                 │  │
│  │         └─ FinalizedDeathJob (Parallel)                 │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         FinishInitializeSystem (OrderLast)                │  │
│  │         - Deshabilita componentes Initialize              │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         GameCameraSystem                                  │  │
│  │         - Actualiza posición y rotación de cámara        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│         ↓ EndSimulationEntityCommandBufferSystem              │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                  PresentationSystemGroup                        │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         MainCameraSystem                                   │  │
│  │         - Actualiza cámara principal para renderizado     │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Flujo de Datos: Inicialización

```
Config (Singleton)
    ↓
GameInitializeSystem
    ├─→ SetupTeams
    │   ├─→ Crea TeamManager entities
    │   └─→ Crea planetas home
    │
    ├─→ SpawnNeutralPlanets
    │   └─→ Crea planetas neutrales
    │
    ├─→ SpawnMoons
    │   └─→ Crea lunas alrededor de planetas
    │
    ├─→ SpawnInitialShips
    │   └─→ Crea naves iniciales
    │
    ├─→ SpawnInitialTeamBuildings
    │   └─→ Crea edificios iniciales
    │
    ├─→ CreateTargetablesSpatialDatabase
    │   └─→ Inicializa base de datos espacial
    │
    ├─→ CreatePlanetNavigationGrid
    │   └─→ Crea grid de navegación
    │
    └─→ ComputePlanetsNetwork
        └─→ Calcula red de planetas
```

## Flujo de Datos: Ciclo de Frame

```
Frame Inicio
    ↓
┌─────────────────────────────────────┐
│  TeamAISystem                       │
│                                     │
│  1. Evaluar flotas                 │
│  2. Analizar planetas              │
│  3. Generar acciones                │
│     ├─→ FighterAction[]            │
│     ├─→ WorkerAction[]             │
│     ├─→ TraderAction[]             │
│     └─→ FactoryAction[]            │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  ApplyTeamSystem                    │
│  - Aplica decisiones de IA         │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  BuildSpatialDatabaseGroup         │
│  - Limpia y reconstruye            │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  PlanetSystem                        │
│                                     │
│  - Evalúa naves alrededor          │
│  - Actualiza recursos              │
│  - Maneja conversión                │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  BuildingSystem                    │
│                                     │
│  - Construye edificios             │
│  - Actualiza torretas              │
│  - Produce naves                   │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  ShipSystem                        │
│                                     │
│  - Navegación                      │
│  - IA (lee acciones generadas)     │
│  - Combate                         │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  LaserSystem                       │
│  - Actualiza proyectiles            │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  TransformSystemGroup              │
│  - Actualiza transformaciones      │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  VFXSystem                         │
│  - Procesa efectos visuales         │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  DeathSystem                       │
│  - Destruye entidades muertas      │
└─────────────────────────────────────┘
    ↓
Frame Fin
```

## Flujo de IA: Toma de Decisiones

```
TeamAISystem
    ↓
┌─────────────────────────────────────┐
│  Evaluación de Flotas              │
│  ├─→ FighterFleetAssessmentJob     │
│  ├─→ WorkerFleetAssessmentJob      │
│  └─→ TraderFleetAssessmentJob      │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Análisis de Planetas              │
│  - Calcula amenaza                 │
│  - Calcula seguridad               │
│  - Evalúa recursos                 │
│  - Calcula distancia               │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Generación de Acciones            │
│                                     │
│  FighterAction[]                    │
│  ├─→ Planeta para atacar           │
│  └─→ Planeta para defender         │
│                                     │
│  WorkerAction[]                     │
│  ├─→ Planeta para capturar         │
│  └─→ Edificio para construir       │
│                                     │
│  TraderAction[]                     │
│  └─→ Ruta comercial                │
│                                     │
│  FactoryAction[]                    │
│  └─→ Tipo de nave a producir       │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Asignación de Importancia         │
│  - Sistema de utilidades            │
│  - Weighted random                  │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  ApplyTeamSystem                    │
│  - Aplica acciones a entidades      │
└─────────────────────────────────────┘
    ↓
Sistemas Específicos
├─→ ShipSystem (ejecuta acciones)
├─→ BuildingSystem (ejecuta acciones)
└─→ PlanetSystem (actualiza estado)
```

## Flujo de Combate

```
FighterAIJob (Parallel)
    ↓
┌─────────────────────────────────────┐
│  Detección de Objetivos            │
│  - Usa SpatialDatabase              │
│  - Filtra por equipo enemigo        │
│  - Calcula distancia                │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Selección de Acción                │
│  - Lee FighterAction[]              │
│  - Aplica bias de proximidad        │
│  - Weighted random                  │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Actualización de Timers           │
│  - AttackTimer                      │
│  - DetectionTimer                    │
└─────────────────────────────────────┘
    ↓
    ¿Listo para atacar?
    ├─→ Sí → Habilita ExecuteAttack
    └─→ No → Continúa
    ↓
FighterExecuteAttackJob (Single-threaded)
    ↓
┌─────────────────────────────────────┐
│  Ejecución de Ataque                │
│  - Aplica daño                      │
│  - Crea efectos visuales            │
│  - Consume recursos si necesario    │
└─────────────────────────────────────┘
    ↓
LaserSystem
    ↓
┌─────────────────────────────────────┐
│  Proyectiles Láser                 │
│  - Actualiza posición               │
│  - Detecta colisiones              │
│  - Aplica daño                     │
└─────────────────────────────────────┘
```

## Flujo de Navegación

```
ShipNavigationJob (Parallel)
    ↓
┌─────────────────────────────────────┐
│  Obtiene Objetivo                   │
│  - NavigationTargetEntity           │
│  - NavigationTargetPosition         │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Evasión de Planetas                │
│  - Usa PlanetNavigationGrid         │
│  - Calcula celda actual             │
│  - Obtiene planeta más cercano      │
│  - Calcula dirección de evasión      │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Cálculo de Dirección              │
│  - Hacia objetivo                   │
│  - Con evasión de planetas          │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  Actualización de Movimiento       │
│  - Calcula velocidad                │
│  - Actualiza posición               │
│  - Verifica llegada a destino       │
└─────────────────────────────────────┘
```

## Dependencias Críticas

```
GameInitializeSystem
    └─→ Requiere: Config, ShipCollection, BuildingCollection
    
TeamAISystem
    └─→ Requiere: Config, TeamManagerReference
    
ShipSystem
    └─→ Requiere:
        ├─→ BuildSpatialDatabaseGroup (completado)
        ├─→ TeamAISystem (completado)
        ├─→ PlanetNavigationGrid (disponible)
        └─→ SpatialDatabaseSingleton
    
PlanetSystem
    └─→ Requiere:
        ├─→ BuildSpatialDatabaseGroup (completado)
        └─→ SpatialDatabaseSingleton
    
BuildingSystem
    └─→ Requiere:
        ├─→ PlanetSystem (completado)
        ├─→ BuildSpatialDatabaseGroup (completado)
        └─→ SpatialDatabaseSingleton
    
DeathSystem
    └─→ Requiere: VFXExplosionsSingleton, VFXThrustersSingleton
    
Todos los sistemas
    └─→ Requieren: Config (singleton)
```

