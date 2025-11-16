# Documentación ECS Galaxy

Bienvenido a la documentación del proyecto ECS Galaxy. Esta documentación está organizada para cubrir tanto aspectos del juego como detalles técnicos de implementación.

## 📚 Índice de Documentación

### Documentación General

1. **[Game Overview](./game-overview.md)**
   - Descripción del juego y mecánicas
   - Tipos de naves (Fighters, Workers, Traders)
   - Tipos de edificios (Factories, Turrets, Research)
   - Sistema de planetas y recursos

### Documentación Técnica

2. **[Code Overview](./code-overview.md)**
   - Visión general del código
   - Inicialización del juego
   - Setup de autoría
   - Sistemas principales (Planetas, Edificios, Naves, IA)
   - Estructuras de aceleración
   - Sistema VFX
   - LODs

3. **[Technical Architecture](./technical-architecture.md)** ⭐ **NUEVO**
   - Visión general de la arquitectura ECS
   - Estructura de directorios detallada
   - Sistemas principales y sus responsabilidades
   - Flujo de ejecución completo
   - Optimizaciones implementadas
   - Patrones de diseño utilizados
   - Consideraciones de rendimiento

4. **[Systems Reference](./systems-reference.md)** ⭐ **NUEVO**
   - Documentación detallada de cada sistema
   - Jobs y sus responsabilidades
   - Orden de ejecución
   - Dependencias entre sistemas
   - Componentes requeridos

5. **[Systems Flow](./systems-flow.md)** ⭐ **NUEVO**
   - Diagrama de flujo de ejecución visual
   - Flujo de datos entre sistemas
   - Flujo de IA y toma de decisiones
   - Flujo de combate
   - Flujo de navegación
   - Dependencias críticas

6. **[Development Guide](./development-guide.md)** ⭐ **NUEVO**
   - Configuración del entorno de desarrollo
   - Guía para agregar nuevos componentes
   - Guía para crear nuevos sistemas
   - Guía para agregar nuevos tipos de naves
   - Guía para agregar nuevos edificios
   - Convenciones de código
   - Mejores prácticas
   - Guía de debugging y testing

### Debugging y Herramientas

7. **[Debug Views](./debug-views.md)**
   - Visualizaciones de debug disponibles
   - Fighter Actions
   - Worker Actions
   - Trader Actions
   - Spatial Database
   - Planet Navigation Grid
   - Planet Network

## 🚀 Inicio Rápido

Si eres nuevo en el proyecto, te recomendamos seguir este orden:

1. Lee **[Game Overview](./game-overview.md)** para entender el juego
2. Revisa **[Code Overview](./code-overview.md)** para entender la estructura del código
3. Estudia **[Technical Architecture](./technical-architecture.md)** para entender la arquitectura
4. Consulta **[Systems Reference](./systems-reference.md)** cuando trabajes con sistemas específicos
5. Usa **[Development Guide](./development-guide.md)** cuando quieras extender el proyecto
6. Revisa **[Systems Flow](./systems-flow.md)** para entender cómo interactúan los sistemas

## 🔍 Búsqueda por Tema

### Arquitectura y Diseño
- **[Technical Architecture](./technical-architecture.md)** - Arquitectura completa
- **[Systems Flow](./systems-flow.md)** - Flujo de ejecución
- **[Code Overview](./code-overview.md)** - Estructura del código

### Sistemas Específicos
- **[Systems Reference](./systems-reference.md)** - Referencia completa
- **[Code Overview](./code-overview.md)** - Visión general de sistemas

### Desarrollo y Extensión
- **[Development Guide](./development-guide.md)** - Guía completa de desarrollo
- **[Code Overview](./code-overview.md)** - Setup de autoría

### Debugging
- **[Debug Views](./debug-views.md)** - Visualizaciones de debug
- **[Development Guide](./development-guide.md)** - Sección de debugging

## 📖 Información Adicional

### Conceptos Clave

- **ECS (Entity Component System)**: Paradigma arquitectónico utilizado
- **DOTS**: Data-Oriented Technology Stack de Unity
- **Jobs**: Sistema de paralelización de Unity
- **Burst**: Compilador optimizador de Unity
- **Blob Assets**: Datos compartidos e inmutables

### Recursos Externos

- [Unity ECS Documentation](https://docs.unity3d.com/Packages/com.unity.entities@latest)
- [DOTS Best Practices](https://docs.unity3d.com/Packages/com.unity.entities@latest/manual/index.html)
- [Burst Compiler](https://docs.unity3d.com/Packages/com.unity.burst@latest)

## 🤝 Contribuir

Si planeas contribuir al proyecto:

1. Lee **[Development Guide](./development-guide.md)**
2. Revisa las convenciones de código
3. Sigue las mejores prácticas establecidas
4. Documenta tus cambios

## 📝 Notas

- Los documentos marcados con ⭐ **NUEVO** son adiciones recientes a la documentación
- La documentación se actualiza continuamente
- Si encuentras información desactualizada, por favor abre un issue

---

**Última actualización**: 2024

