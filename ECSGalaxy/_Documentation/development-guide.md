# Guía de Desarrollo

## Tabla de Contenidos

1. [Configuración del Entorno](#configuración-del-entorno)
2. [Estructura del Código](#estructura-del-código)
3. [Agregar Nuevos Componentes](#agregar-nuevos-componentes)
4. [Crear Nuevos Sistemas](#crear-nuevos-sistemas)
5. [Agregar Nuevas Naves](#agregar-nuevas-naves)
6. [Agregar Nuevos Edificios](#agregar-nuevos-edificios)
7. [Convenciones de Código](#convenciones-de-código)
8. [Mejores Prácticas](#mejores-prácticas)
9. [Debugging](#debugging)
10. [Testing](#testing)

---

## Configuración del Entorno

### Requisitos

- Unity 2022.3 LTS o superior
- Packages DOTS:
  - Entities
  - Collections
  - Mathematics
  - Transforms
  - Burst
  - Jobs

### Configuración Inicial

1. Clona el repositorio
2. Abre el proyecto en Unity
3. Unity importará automáticamente los packages necesarios
4. Abre la escena `Main` para comenzar

---

## Estructura del Código

### Organización de Directorios

```
Assets/Scripts/
├── Components/      # Componentes ECS puros (IComponentData, IBufferElementData)
├── Authoring/       # MonoBehaviours y Bakers para autoría en el editor
├── UI/              # Sistema de interfaz de usuario
└── Utilities/       # Utilidades, helpers y estructuras de datos
```

### Convenciones de Nombres

- **Componentes**: PascalCase (`Ship`, `Planet`, `Health`)
- **Sistemas**: PascalCase + "System" (`ShipSystem`, `PlanetSystem`)
- **Jobs**: PascalCase + "Job" (`ShipNavigationJob`, `FighterAIJob`)
- **Campos privados**: camelCase con prefijo `_` (`_spatialDatabasesQuery`)
- **Campos públicos**: PascalCase

---

## Agregar Nuevos Componentes

### Componente de Datos Simple

```csharp
using Unity.Entities;

public struct MyComponent : IComponentData
{
    public float Value;
    public int Count;
}
```

### Componente de Buffer

```csharp
using Unity.Entities;

[InternalBufferCapacity(0)]  // o un número específico
public struct MyBufferElement : IBufferElementData
{
    public Entity Entity;
    public float Importance;
}
```

### Blob Asset

Para datos compartidos e inmutables:

1. Define la estructura:
```csharp
public struct MyData
{
    public float Speed;
    public float Damage;
    public float Range;
}
```

2. Crea el autor:
```csharp
using UnityEngine;

[CreateAssetMenu]
public class MyDataObject : ScriptableObject, IBlobAuthoring<MyData>
{
    public float Speed;
    public float Damage;
    public float Range;

    public BlobAssetReference<MyData> BakeToBlobData()
    {
        var builder = new BlobBuilder(Allocator.Temp);
        ref var data = ref builder.ConstructRoot<MyData>();
        
        data.Speed = Speed;
        data.Damage = Damage;
        data.Range = Range;
        
        var blobReference = builder.CreateBlobAssetReference<MyData>(Allocator.Persistent);
        builder.Dispose();
        return blobReference;
    }
}
```

3. Usa en el componente:
```csharp
public struct MyComponent : IComponentData
{
    public BlobAssetReference<MyData> Data;
}
```

---

## Crear Nuevos Sistemas

### Sistema Básico

```csharp
using Unity.Burst;
using Unity.Entities;

[BurstCompile]
public partial struct MySystem : ISystem
{
    [BurstCompile]
    public void OnCreate(ref SystemState state)
    {
        // Configuración inicial
        state.RequireForUpdate<Config>();
    }

    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // Lógica del sistema
    }
}
```

### Sistema con Jobs Paralelos

```csharp
[BurstCompile]
public partial struct MySystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        MyJob job = new MyJob
        {
            DeltaTime = SystemAPI.Time.DeltaTime,
        };
        
        state.Dependency = job.ScheduleParallel(state.Dependency);
    }
}

[BurstCompile]
public partial struct MyJob : IJobEntity
{
    public float DeltaTime;

    void Execute(Entity entity, ref MyComponent component)
    {
        // Lógica del job
    }
}
```

### Orden de Ejecución

Especifica el orden usando atributos:

```csharp
[UpdateAfter(typeof(SomeSystem))]
[UpdateBefore(typeof(OtherSystem))]
[UpdateInGroup(typeof(SomeGroup))]
public partial struct MySystem : ISystem
{
    // ...
}
```

---

## Agregar Nuevas Naves

### 1. Crear el Componente

```csharp
public struct MyShipType : IComponentData
{
    public float SpecialAbility;
    public BlobAssetReference<MyShipTypeData> ShipData;
}
```

### 2. Crear Blob Asset Data

```csharp
public struct MyShipTypeData
{
    public float MaxSpeed;
    public float Acceleration;
    public float Health;
    // ... más propiedades
}
```

### 3. Crear el Authoring

```csharp
using UnityEngine;

[RequireComponent(typeof(ShipAuthoring))]
public class MyShipTypeAuthoring : MonoBehaviour
{
    public MyShipTypeDataObject ShipData;
    
    class Baker : Baker<MyShipTypeAuthoring>
    {
        public override void Bake(MyShipTypeAuthoring authoring)
        {
            Entity entity = GetEntity(TransformUsageFlags.Dynamic);
            
            AddComponent(entity, new MyShipType
            {
                ShipData = authoring.ShipData.BakeToBlobData(),
            });
        }
    }
}
```

### 4. Agregar Lógica en ShipSystem

```csharp
[BurstCompile]
public partial struct MyShipTypeJob : IJobEntity
{
    public float DeltaTime;

    void Execute(Entity entity, ref MyShipType shipType, ref Ship ship)
    {
        // Lógica específica del tipo de nave
    }
}
```

### 5. Integrar en ShipSystem.OnUpdate

```csharp
MyShipTypeJob myShipTypeJob = new MyShipTypeJob
{
    DeltaTime = SystemAPI.Time.DeltaTime,
};
state.Dependency = myShipTypeJob.ScheduleParallel(state.Dependency);
```

---

## Agregar Nuevos Edificios

Similar a las naves, pero usando `BuildingSystem`:

### 1. Crear el Componente

```csharp
public struct MyBuilding : IComponentData
{
    public float SpecialProperty;
    public BlobAssetReference<MyBuildingData> BuildingData;
}
```

### 2. Crear Authoring

```csharp
[RequireComponent(typeof(HealthAuthoring))]
[RequireComponent(typeof(TeamAuthoring))]
public class MyBuildingAuthoring : MonoBehaviour
{
    public MyBuildingDataObject BuildingData;
    
    class Baker : Baker<MyBuildingAuthoring>
    {
        public override void Bake(MyBuildingAuthoring authoring)
        {
            Entity entity = GetEntity(TransformUsageFlags.Dynamic);
            
            AddComponent(entity, new MyBuilding
            {
                BuildingData = authoring.BuildingData.BakeToBlobData(),
            });
        }
    }
}
```

### 3. Agregar Jobs en BuildingSystem

---

## Convenciones de Código

### Uso de Attributes

- `[BurstCompile]` en todos los sistemas y jobs críticos
- `[UpdateAfter/UpdateBefore]` para orden de ejecución
- `[RequireComponent]` en authorings cuando hay dependencias

### Gestión de Memoria

```csharp
// Usar Allocator.Temp para datos temporales del frame
NativeArray<float> tempData = new NativeArray<float>(count, Allocator.Temp);
// ... usar
tempData.Dispose();

// Usar Allocator.TempJob para datos en jobs
NativeList<int> jobData = new NativeList<int>(Allocator.TempJob);
// Disposed automáticamente cuando el job termina
```

### Queries

```csharp
// Usar SystemAPI.QueryBuilder para queries
EntityQuery query = SystemAPI.QueryBuilder()
    .WithAll<Ship, Health>()
    .WithNone<Death>()
    .Build();

// O usar partial struct para queries generadas
public partial struct MySystem : ISystem
{
    [BurstCompile]
    public void OnUpdate(ref SystemState state)
    {
        // Query generada automáticamente
        foreach (var (ship, health) in SystemAPI.Query<RefRO<Ship>, RefRW<Health>>())
        {
            // ...
        }
    }
}
```

---

## Mejores Prácticas

### 1. Rendimiento

- ✅ Usa `BurstCompile` en sistemas críticos
- ✅ Paraleliza con `ScheduleParallel` cuando sea posible
- ✅ Evita allocations innecesarios
- ✅ Usa `NativeArray` y `NativeList` para datos temporales
- ✅ Caché `ComponentLookup` dentro de jobs

### 2. Organización

- ✅ Mantén sistemas enfocados en una responsabilidad
- ✅ Separa lógica paralela de lógica single-threaded
- ✅ Usa jobs anidados para lógica compleja

### 3. Debugging

- ✅ Usa `Debug.Log` solo fuera de código Burst
- ✅ Usa Profiler para identificar cuellos de botella
- ✅ Usa Entity Debugger de Unity para inspeccionar entidades

### 4. Testing

- ✅ Testa utilidades de forma aislada
- ✅ Usa unit tests para lógica matemática
- ✅ Testa integración de sistemas críticos

---

## Debugging

### Entity Debugger

Unity proporciona un Entity Debugger que permite:
- Ver todas las entidades
- Inspeccionar componentes
- Ver queries activas
- Profiling de sistemas

### Profiling

1. Abre **Window > Analysis > Profiler**
2. Selecciona **Unity Profiler**
3. Haz play y ejecuta la simulación
4. Analiza tiempos de sistemas

### Debug Visualizations

El proyecto incluye visualizaciones de debug:
- Espacial Database
- Planet Navigation Grid
- Planet Network
- Acciones de IA

Ver [Debug Views](./debug-views.md) para más detalles.

### Logging

```csharp
// Solo fuera de código Burst
#if UNITY_EDITOR
    Debug.Log($"Entity {entity} has value {value}");
#endif
```

---

## Testing

### Unit Tests

Crea tests en `Tests/`:

```csharp
using NUnit.Framework;
using Unity.Mathematics;

public class MathUtilitiesTests
{
    [Test]
    public void CalculateDistance_ReturnsCorrectValue()
    {
        float3 a = new float3(0, 0, 0);
        float3 b = new float3(1, 0, 0);
        float distance = math.distance(a, b);
        
        Assert.AreEqual(1f, distance);
    }
}
```

### Integration Tests

Para sistemas completos:

```csharp
using NUnit.Framework;
using Unity.Entities;

public class ShipSystemTests
{
    [Test]
    public void ShipNavigation_UpdatesPosition()
    {
        // Crear world
        World world = new World("TestWorld");
        
        // Crear entidades de prueba
        // Ejecutar sistemas
        // Verificar resultados
        
        world.Dispose();
    }
}
```

---

## Recursos Adicionales

- [Unity ECS Documentation](https://docs.unity3d.com/Packages/com.unity.entities@latest)
- [DOTS Best Practices](https://docs.unity3d.com/Packages/com.unity.entities@latest/manual/index.html)
- [Burst Documentation](https://docs.unity3d.com/Packages/com.unity.burst@latest)
- [Arquitectura Técnica](./technical-architecture.md)
- [Referencia de Sistemas](./systems-reference.md)

---

## Preguntas Frecuentes

### ¿Cómo agrego un nuevo tipo de recurso?

1. Extiende `float3` ResourceGenerationRate y ResourceStorage a un nuevo struct
2. Actualiza `Planet` component
3. Actualiza sistemas que usan recursos

### ¿Cómo creo una nueva acción de IA?

1. Crea un nuevo `BufferElementData` para la acción
2. Agrega lógica en `TeamAISystem` para generar acciones
3. Agrega lógica en el sistema correspondiente para ejecutar acciones

### ¿Cómo optimizo un sistema lento?

1. Usa Profiler para identificar cuellos de botella
2. Paraleliza con Jobs cuando sea posible
3. Reduce allocations
4. Usa estructuras espaciales para queries eficientes
5. Considera Burst Compile si no está habilitado

---

## Contribuir

Al contribuir al proyecto:

1. Sigue las convenciones de código establecidas
2. Documenta cambios significativos
3. Testa tus cambios
4. Actualiza documentación si es necesario
5. Crea pull requests descriptivos

