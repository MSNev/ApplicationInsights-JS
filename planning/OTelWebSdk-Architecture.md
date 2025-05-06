# OpenTelemetry Web SDK Architecture

## Overview

The OpenTelemetry Web SDK is designed as a modern, modular implementation that follows the OpenTelemetry specification. It provides a complete tracing, logging, and metrics solution for web applications without relying on global singletons.

## Design Principles

1. **Interface-First Design**: All public APIs are defined as interfaces
2. **Closure-Based Implementation**: Internal implementations use closures instead of classes
3. **Factory Functions**: Helper creation functions are used instead of exposing constructors
4. **No Global Singletons**: No global context, api, trace, or log instances
5. **Minimal API Surface**: Only necessary methods and properties are exposed
6. **Tree-Shakable**: Support for tree-shaking to minimize bundle sizes

## High-level Architecture Diagram

```
┌───────────────────────────────────────────────────────────┐
│                       OTelWebSdk                          │
├───────────┬───────────────┬─────────────┬────────────────┐
│ Trace     │ Log           │ Metric      │ Context        │
│ Provider  │ Provider      │ Provider    │ Manager        │
├───────────┼───────────────┼─────────────┼────────────────┤
│ Tracers   │ Loggers       │ Meters      │ Baggage        │
└───────────┴───────────────┴─────────────┴────────────────┘
              ▲                   ▲
              │                   │
┌─────────────┴───────────────────┴────────────────────────┐
│                Telemetry Processing Pipeline              │
└─────────────────────────────┬────────────────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────┐
│                       Exporters                         │
│   (Azure Monitor, Console, OTLP, Custom, etc.)          │
└─────────────────────────────────────────────────────────┘
```

## Closure Pattern with DynamicProto

Instead of using traditional ES6 classes, the SDK uses closures combined with DynamicProto-JS to optimize performance and reduce bundle size. This approach has several advantages:

1. **Better Minification**: Closure variables can be minified more effectively
2. **Private State**: State variables remain truly private
3. **Performance**: Reduced prototype chain lookups
4. **Bundle Size**: Smaller final bundle size after tree-shaking
5. **Compatibility**: Better compatibility with older browsers

### Example Pattern

```typescript
export function createExample(config: IExampleConfig): IExample {
  // Private closure variables
  let _internalState = { count: 0 };
  let _config = config;

  // Create the instance
  let _self = {} as IExample;

  // Define the "public" methods using DynamicProto
  dynamicProto(/* placeholder for the constructor */ {}, _self, (self) => {
    // Public method implementation
    self.publicMethod = () => {
      _internalState.count++;
      return _internalState.count;
    };
  });

  return _self;
}
```

## Interface Naming Conventions

The SDK follows strict naming conventions for interfaces:

- **Public Interfaces**: Prefixed with `I` (e.g., `ITraceProvider`)
- **Internal Interfaces**: Prefixed with `_I` (e.g., `_ITraceProviderInternal`)
- **JSDoc Tags**: All internal interfaces are marked with `@internal` JSDoc tags

## Module Structure

The SDK is organized into the following module structure:

```
otel-web-sdk/
├── core/
│   ├── interfaces/
│   ├── implementation/
│   └── index.ts
├── trace/
│   ├── interfaces/
│   ├── implementation/
│   └── index.ts
├── log/
│   ├── interfaces/
│   ├── implementation/
│   └── index.ts
├── metric/
│   ├── interfaces/
│   ├── implementation/
│   └── index.ts
├── context/
│   ├── interfaces/
│   ├── implementation/
│   └── index.ts
├── export/
│   ├── interfaces/
│   ├── implementation/
│   └── index.ts
└── index.ts
```

Each module exposes only its public interfaces and factory functions, keeping implementation details hidden.

## Configuration Management

Configuration in the SDK follows these principles:

1. **Default Configuration**: Sensible defaults for all options
2. **Overridable**: All configuration options can be overridden
3. **Hierarchical**: Configuration can be provided at different levels (SDK, provider, tracer, etc.)
4. **Dynamic**: Configuration can be updated at runtime
5. **Validation**: All configuration is validated before use

## Error Handling

The SDK uses a consistent error handling approach:

1. **Error Callbacks**: Errors are reported to registered error callbacks
2. **Graceful Degradation**: The SDK continues to operate even in error conditions
3. **Detailed Error Information**: Errors include detailed context information
4. **Error Categories**: Errors are categorized for easier filtering

## Next Steps

For detailed implementation specifications, refer to the following documents:

- [OTelWebSdk-Core.md](./OTelWebSdk-Core.md)
- [OTelWebSdk-Trace.md](./OTelWebSdk-Trace.md)
- [OTelWebSdk-Log.md](./OTelWebSdk-Log.md)
- [OTelWebSdk-Metric.md](./OTelWebSdk-Metric.md)
- [OTelWebSdk-Context.md](./OTelWebSdk-Context.md)
