# Referencia de Sistemas

## Tabla de Contenidos

1. [Sistemas de Inicialización](#sistemas-de-inicialización)
2. [Sistemas de IA](#sistemas-de-ia)
3. [Sistemas de Naves](#sistemas-de-naves)
4. [Sistemas de Planetas](#sistemas-de-planetas)
5. [Sistemas de Edificios](#sistemas-de-edificios)
6. [Sistemas de Base de Datos Espacial](#sistemas-de-base-de-datos-espacial)
7. [Sistemas de Muerte](#sistemas-de-muerte)
8. [Sistemas Visuales](#sistemas-visuales)
9. [Sistemas de UI](#sistemas-de-ui)

---

## Sistemas de Inicialización

### GameInitializeSystem

**Ubicación**: `Assets/Scripts/GameInitializeSystem.cs`

**Grupo**: `BeginSimulationMainThreadGroup`

**Responsabilidad**: Inicializa todo el mundo del juego una vez al inicio.

**Métodos Principales**:
- `SetupTeams`: Crea equipos y planetas home
- `SpawnNeutralPlanets`: Spawnea planetas neutrales
- `SpawnMoons`: Crea lunas alrededor de planetas
- `SpawnInitialShips`: Spawnea naves iniciales
- `SpawnInitialTeamBuildings`: Spawnea edificios iniciales
- `CreateTargetablesSpatialDatabase`: Construye base de datos espacial
- `CreatePlanetNavigationGrid`: Construye grid de navegación
- `ComputePlanetsNetwork`: Calcula red de planetas conectados

**Componentes Requeridos**:
- `Config`
- `ShipCollection`
- `BuildingCollection`
- `ShipSpawnParams`
- `GameCamera`

---

### SimulationRateSystem

**Ubicación**: `Assets/Scripts/SimulationRateSystem.cs`

**Grupos**: `InitializationSystemGroup`, `SimulationSystemGroup` (OrderFirst)

**Responsabilidad**: Controla la velocidad de simulación.

**Características**:
- Permite velocidad de simulación configurable
- Soporte para tiempo fijo o escalado
- Actualiza `DeltaTime` global

---

### FinishInitializeSystem

**Ubicación**: `Assets/Scripts/FinishInitializeSystem.cs`

**Grupo**: `SimulationSystemGroup` (OrderLast)

**Responsabilidad**: Finaliza la inicialización deshabilitando componentes `Initialize`.

**Proceso**:
1. Itera todas las entidades con `Initialize` habilitado
2. Deshabilita el componente
3. Permite múltiples sistemas inicializar en el mismo frame

---

## Sistemas de IA

### TeamAISystem

**Ubicación**: `Assets/Scripts/TeamAISystem.cs`

**Ubicación**: `UpdateAfter(BeginSimulationMainThreadGroup)`

**Responsabilidad**: Sistema principal de IA que evalúa estrategias y genera acciones para cada equipo.

**Proceso**:

1. **Evaluación de Flotas**:
   - `FighterFleetAssessmentJob`: Cuenta fighters por equipo
   - `WorkerFleetAssessmentJob`: Cuenta workers por equipo
   - `TraderFleetAssessmentJob`: Cuenta traders por equipo

2. **Análisis de Planetas**:
   - Calcula amenaza (`ThreatLevel`)
   - Calcula seguridad (`SafetyLevel`)
   - Evalúa recursos disponibles
   - Calcula distancia desde planetas capturados

3. **Generación de Acciones**:
   - `FighterAction`: Planetas para atacar/defender
   - `WorkerAction`: Planetas para capturar o edificios para construir
   - `TraderAction`: Planetas para intercambio de recursos
   - `FactoryAction`: Tipos de naves para producir

4. **Asignación de Importancia**:
   - Sistema de utilidades basado en factores:
     - Amenaza/Seguridad
     - Recursos
     - Distancia
     - Estado del equipo

**Componentes Requeridos**:
- `Config`
- `TeamManagerReference`

---

### ApplyTeamSystem

**Ubicación**: `Assets/Scripts/ApplyTeamSystem.cs`

**Ubicación**: `UpdateAfter(BeginSimulationMainThreadGroup)`

**Responsabilidad**: Aplica decisiones de IA a las entidades del juego.

**Proceso**:
- Itera entidades con `ApplyTeam` habilitado
- Aplica colores y propiedades de equipo
- Deshabilita `ApplyTeam` después de aplicar

---

## Sistemas de Naves

### ShipSystem

**Ubicación**: `Assets/Scripts/ShipSystem.cs`

**Orden de Ejecución**: 
- `UpdateAfter(BuildSpatialDatabaseGroup)`
- `UpdateAfter(TeamAISystem)`
- `UpdateAfter(ApplyTeamSystem)`
- `UpdateBefore(DeathSystem)`
- `UpdateBefore(TransformSystemGroup)`

**Responsabilidad**: Maneja toda la lógica de naves incluyendo navegación, IA y combate.

**Jobs Principales**:

#### ShipInitializeJob
- **Tipo**: `IJobEntity`
- **Responsabilidad**: Inicializa naves nuevas
- **Acciones**:
  - Configura propulsores VFX
  - Inicializa velocidades
  - Configura datos iniciales

#### FighterInitializeJob
- **Tipo**: `IJobEntity`
- **Responsabilidad**: Inicialización específica de fighters
- **Acciones**:
  - Resetea timers de ataque/detección
  - Configura datos de combate

#### ShipNavigationJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: Navegación y movimiento de todas las naves
- **Características**:
  - Usa `PlanetNavigationGrid` para evasión de planetas
  - Calcula dirección hacia objetivo
  - Actualiza velocidad y posición
  - Maneja llegada a destino

#### FighterAIJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: IA de combate para fighters
- **Proceso**:
  1. Detecta objetivos enemigos usando `SpatialDatabase`
  2. Evalúa acciones disponibles (`FighterAction`)
  3. Selecciona acción con weighted random
  4. Actualiza timers de ataque y detección
  5. Habilita `ExecuteAttack` cuando está listo

#### WorkerAIJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: IA para workers
- **Proceso**:
  1. Evalúa acciones disponibles (`WorkerAction`)
  2. Selecciona acción (captura o construcción)
  3. Habilita `ExecutePlanetCapture` o `ExecuteBuild`

#### TraderAIJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: IA para traders
- **Proceso**:
  1. Evalúa acciones disponibles (`TraderAction`)
  2. Selecciona ruta comercial
  3. Habilita `ExecuteTrade`

#### FighterExecuteAttackJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Ejecuta ataques de fighters
- **Acciones**:
  - Aplica daño a objetivos
  - Crea efectos visuales (chispas de láser)
  - Consume recursos del planeta si es necesario

#### WorkerExecutePlanetCaptureJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Ejecuta captura de planetas
- **Proceso**:
  - Actualiza progreso de captura
  - Cambia equipo del planeta cuando se completa

#### WorkerExecuteBuildJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Ejecuta construcción de edificios
- **Proceso**:
  - Actualiza progreso de construcción
  - Completa construcción cuando está listo

#### TraderExecuteTradeJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Ejecuta intercambio de recursos
- **Proceso**:
  - Transfiere recursos entre planetas
  - Actualiza cargas de traders

**Componentes Requeridos**:
- `Config`
- `SpatialDatabaseSingleton`
- `PlanetNavigationGrid`
- `TeamManagerReference`
- `BeginSimulationEntityCommandBufferSystem.Singleton`
- `VFXHitSparksSingleton`
- `VFXThrustersSingleton`

---

## Sistemas de Planetas

### PlanetSystem

**Ubicación**: `Assets/Scripts/PlanetSystem.cs`

**Orden de Ejecución**:
- `UpdateAfter(BuildSpatialDatabaseGroup)`
- `UpdateBefore(BuildingSystem)`

**Responsabilidad**: Maneja lógica de planetas incluyendo recursos, conversión y evaluación de naves.

**Jobs**:

#### PlanetShipsAssessmentJob
- **Tipo**: `IJobEntity` + `IJobEntityChunkBeginEnd`
- **Responsabilidad**: Evalúa naves alrededor de cada planeta
- **Proceso**:
  - Usa `SpatialDatabase` para encontrar naves cercanas
  - Cuenta naves por tipo y equipo
  - Actualiza `PlanetShipsAssessment` buffer
  - Distribuye evaluación entre frames para rendimiento

#### PlanetConversionJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: Maneja conversión/captura de planetas
- **Proceso**:
  - Actualiza progreso de captura
  - Cambia equipo cuando se completa
  - Resetea progreso si captura falla

#### PlanetResourcesJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: Genera recursos en planetas
- **Proceso**:
  - Incrementa recursos basado en `ResourceGenerationRate`
  - Limita a `ResourceMaxStorage`

#### PlanetClearBuildingsDataJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: Limpia bonificaciones de edificios
- **Proceso**:
  - Resetea `ResearchBonuses` al inicio del frame
  - Permite que edificios de investigación apliquen bonificaciones

**Componentes Requeridos**:
- `Config`
- `SpatialDatabaseSingleton`
- `BeginSimulationEntityCommandBufferSystem.Singleton`
- `TeamManagerReference`

---

## Sistemas de Edificios

### BuildingSystem

**Ubicación**: `Assets/Scripts/BuildingSystem.cs`

**Orden de Ejecución**:
- `UpdateAfter(BeginSimulationMainThreadGroup)`
- `UpdateAfter(BuildSpatialDatabaseGroup)`
- `UpdateAfter(PlanetSystem)`
- `UpdateBefore(DeathSystem)`

**Responsabilidad**: Maneja toda la lógica de edificios incluyendo construcción, ataque y producción.

**Jobs**:

#### TurretInitializeJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: Inicializa torretas
- **Acciones**:
  - Configura timers de ataque
  - Inicializa datos de combate

#### BuildingInitializeJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: Inicializa edificios
- **Acciones**:
  - Configura estados iniciales
  - Inicializa datos específicos del tipo

#### BuildingConstructionJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Actualiza construcción de edificios
- **Proceso**:
  - Incrementa progreso de construcción
  - Completa construcción cuando está listo
  - Consume recursos necesarios

#### TurretUpdateAttackJob
- **Tipo**: `IJobEntity` (Parallel)
- **Responsabilidad**: Actualiza detección de objetivos y timers
- **Proceso**:
  - Usa `SpatialDatabase` para encontrar enemigos en rango
  - Actualiza timers de ataque
  - Habilita `ExecuteAttack` cuando está listo

#### TurretExecuteAttack
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Ejecuta ataques de torretas
- **Proceso**:
  - Aplica daño a objetivos
  - Crea efectos visuales
  - Crea proyectiles láser
  - Consume recursos si es necesario

#### ResearchApplyToPlanetJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Aplica bonificaciones de investigación
- **Proceso**:
  - Lee bonificaciones del edificio
  - Aplica al planeta asociado
  - Actualiza `ResearchBonuses` en `Planet`

#### FactoryJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Produce naves en fábricas
- **Proceso**:
  1. Evalúa acciones disponibles (`FactoryAction`)
  2. Selecciona tipo de nave a producir
  3. Incrementa progreso de producción
  4. Spawnea nave cuando está completo
  5. Consume recursos necesarios

**Componentes Requeridos**:
- `Config`
- `SpatialDatabaseSingleton`
- `ShipCollection`
- `BuildingCollection`
- `BeginSimulationEntityCommandBufferSystem.Singleton`
- `TeamManagerReference`
- `VFXHitSparksSingleton`

---

## Sistemas de Base de Datos Espacial

### BuildSpatialDatabasesSystem

**Ubicación**: `Assets/Scripts/BuildSpatialDatabasesSystem.cs`

**Grupo**: `BuildSpatialDatabaseGroup`

**Responsabilidad**: Construye base de datos espacial para queries rápidas.

**Jobs**:

#### BuildSpatialDatabaseSingleJob
- **Modo**: Single-threaded
- **Proceso**:
  - Itera todas las entidades con `Targetable`
  - Calcula celda espacial
  - Agrega a base de datos

#### SpatialDatabaseParallelComputeCellIndexJob
- **Modo**: Parallel (pre-cálculo)
- **Proceso**:
  - Calcula índices de celdas en paralelo
  - Almacena en `SpatialDatabaseCellIndex`

#### BuildSpatialDatabaseParallelJob
- **Modo**: Parallel
- **Proceso**:
  - Múltiples jobs trabajan en diferentes celdas
  - Cada job procesa sus celdas asignadas
  - Agrega elementos a base de datos

**Características**:
- Soporte para construcción paralela o secuencial
- Configurable via `Config.BuildSpatialDatabaseParallel`
- Optimizado para queries de proximidad

**Componentes Requeridos**:
- `Config`
- `SpatialDatabaseSingleton`

---

### ClearSpatialDatabaseSystem

**Ubicación**: `Assets/Scripts/Utilities/SpatialDatabase/ClearSpatialDatabaseSystem.cs`

**Grupo**: `BuildSpatialDatabaseGroup` (OrderFirst)

**Responsabilidad**: Limpia la base de datos espacial al inicio de cada frame.

**Proceso**:
- Limpia todos los elementos almacenados
- Prepara para reconstrucción

---

## Sistemas de Muerte

### DeathSystem

**Ubicación**: `Assets/Scripts/DeathSystem.cs`

**Grupo**: `SimulationSystemGroup` (OrderLast)

**Orden**: `UpdateBefore(EndSimulationEntityCommandBufferSystem)`

**Responsabilidad**: Maneja muerte de entidades y efectos relacionados.

**Jobs**:

#### BuildingDeathJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Maneja muerte de edificios
- **Proceso**:
  - Limpia referencias de edificios
  - Crea efectos de explosión
  - Actualiza estado de luna

#### ShipDeathJob
- **Tipo**: `IJobEntity` (Single-threaded)
- **Responsabilidad**: Maneja muerte de naves
- **Proceso**:
  - Crea efectos de explosión
  - Mata propulsores VFX
  - Limpia referencias

#### FinalizedDeathJob
- **Tipo**: `IJobEntity` + `IJobEntityChunkBeginEnd` (Parallel)
- **Responsabilidad**: Destruye entidades muertas
- **Proceso**:
  - Itera todas las entidades con `Health.IsDead`
  - Destruye entidades usando `EntityCommandBuffer`

**Componentes Requeridos**:
- `VFXExplosionsSingleton`
- `VFXThrustersSingleton`
- `BeginSimulationEntityCommandBufferSystem.Singleton`

---

## Sistemas Visuales

### VFXSystem

**Ubicación**: `Assets/Scripts/VFXSystem.cs`

**Orden**: `UpdateAfter(ShipPostTransformsSystem)`

**Responsabilidad**: Gestiona efectos visuales eficientemente.

**VFX Managers**:
- `VFXManager<VFXExplosionRequest>`: Explosiones
- `VFXManager<VFXHitSparksRequest>`: Chispas de láser
- `VFXManagerParented<VFXThrusterData>`: Propulsores

**Proceso**:
1. Jobs escriben requests durante el frame
2. `VFXSystem` procesa requests al final del frame
3. Actualiza VFXGraphs via Graphics Buffers
4. VFXGraphs generan partículas

**Características**:
- Un VFXGraph por tipo de efecto
- Sin GameObjects por instancia
- Escalable a miles de efectos

---

### LaserSystem

**Ubicación**: `Assets/Scripts/LaserSystem.cs`

**Orden**:
- `UpdateAfter(BeginSimulationMainThreadGroup)`
- `UpdateBefore(TransformSystemGroup)`

**Responsabilidad**: Maneja proyectiles láser.

**Proceso**:
- Actualiza posición de proyectiles
- Detecta colisiones
- Aplica daño
- Destruye proyectiles que alcanzan destino

---

### GameCameraSystem

**Ubicación**: `Assets/Scripts/GameCameraSystem.cs`

**Grupo**: `SimulationSystemGroup`

**Orden**: 
- `UpdateAfter(TransformSystemGroup)`
- `UpdateBefore(EndSimulationEntityCommandBufferSystem)`

**Responsabilidad**: Actualiza posición y rotación de la cámara del juego.

**Modos**:
- Cámara libre
- Órbita planeta
- Órbita nave

---

### MainCameraSystem

**Ubicación**: `Assets/Scripts/MainCameraSystem.cs`

**Grupo**: `PresentationSystemGroup`

**Responsabilidad**: Actualiza cámara principal para renderizado.

---

## Sistemas de UI

### UISystem

**Ubicación**: `Assets/Scripts/UISystem.cs`

**Responsabilidad**: Puente entre ECS y sistema de UI.

**Proceso**:
- Inicializa UI al inicio
- Maneja eventos de UI
- Actualiza estado de UI desde ECS

**Componentes Requeridos**:
- `BeginSimulationEntityCommandBufferSystem.Singleton`
- `Config`

---

## Resumen de Dependencias

### Orden de Ejecución Crítico

```
1. SimulationRateSystem (OrderFirst)
2. BeginSimulationMainThreadGroup
   ├─ GameInitializeSystem
   └─ BuildSpatialDatabaseGroup
3. TeamAISystem
4. ApplyTeamSystem
5. BuildSpatialDatabaseGroup (reconstrucción)
6. PlanetSystem
7. BuildingSystem
8. ShipSystem
9. LaserSystem
10. TransformSystemGroup
11. VFXSystem
12. DeathSystem (OrderLast)
13. FinishInitializeSystem (OrderLast)
14. GameCameraSystem
```

### Dependencias entre Sistemas

- **ShipSystem** requiere:
  - `BuildSpatialDatabaseGroup` completado
  - `TeamAISystem` completado
  - `PlanetNavigationGrid` disponible

- **BuildingSystem** requiere:
  - `PlanetSystem` completado
  - `BuildSpatialDatabaseGroup` completado

- **PlanetSystem** requiere:
  - `BuildSpatialDatabaseGroup` completado

- Todos los sistemas requieren:
  - `Config` singleton
  - Entity Command Buffer System apropiado

