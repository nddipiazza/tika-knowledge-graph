# Architecture Overview

## Design Philosophy

Apache Tika is designed around several core principles:

1. **Simplicity**: Simple facade API (Tika.java) hides complexity
2. **Extensibility**: Plugin-based architecture for parsers and detectors
3. **Modularity**: Clear separation of concerns across modules
4. **Reliability**: Defensive parsing with error handling
5. **Performance**: Efficient streaming and async processing

## Architectural Layers

### 1. Client Layer
Applications interact with Tika through multiple interfaces:
- **Tika Facade** (`org.apache.tika.Tika`): Simple high-level API
- **Parser API**: Direct parser access for advanced use
- **REST API**: HTTP endpoints (tika-server)
- **gRPC API**: gRPC endpoints (tika-grpc)

### 2. Core Layer (tika-core)
The foundation of Tika, providing:
- **Parser Interface**: Common contract for all parsers
- **Detector Interface**: MIME type detection contract
- **Metadata**: Metadata handling and extraction
- **SAX Handlers**: Content handler implementations
- **Config**: Configuration management

**Key Classes:**
- `org.apache.tika.Tika` - Main facade
- `org.apache.tika.parser.Parser` - Parser interface
- `org.apache.tika.detect.Detector` - Detector interface
- `org.apache.tika.metadata.Metadata` - Metadata container
- `org.apache.tika.config.TikaConfig` - Configuration

### 3. Parser Layer
Two main categories:

#### Standard Parsers (tika-parsers-standard-package)
Common document formats:
- Office documents (DOC, XLS, PPT, DOCX, XLSX, PPTX)
- PDF documents
- HTML, XML, plain text
- Images (JPEG, PNG, TIFF)
- Archives (ZIP, TAR, GZIP)

#### Extended Parsers (tika-parsers-extended)
Specialized formats:
- Scientific formats (NetCDF, HDF, GRIB)
- Database formats (SQLite, Access)
- CAD formats (DWG, DXF)
- Multimedia (audio/video metadata)

### 4. Detection Layer (tika-detectors)
Multiple detection strategies:
- **Magic byte detection**: File header analysis
- **Extension-based**: Filename extension
- **Container detection**: Inspect container formats (ZIP, OLE2)
- **Composite detection**: Combine multiple strategies

### 5. Pipeline Layer (tika-pipes)
Asynchronous batch processing:
- **Fetchers**: Retrieve documents from various sources
- **Emitters**: Output results to various targets
- **Async Processor**: Process documents asynchronously
- **File System Watcher**: Monitor directories for new files

## Design Patterns

### 1. Facade Pattern
`Tika.java` provides a simplified interface to complex subsystems

### 2. Strategy Pattern
Parsers and Detectors are interchangeable strategies

### 3. Composite Pattern
`CompositeParser` aggregates multiple parsers
`CompositeDetector` combines multiple detectors

### 4. Decorator Pattern
Parser wrappers add functionality (error handling, resource limits)

### 5. Service Provider Interface (SPI)
Parsers and Detectors discovered via Java ServiceLoader

### 6. Template Method Pattern
`AbstractParser` provides common parsing structure

## Component Communication

```
Client Code
    │
    ▼
Tika Facade
    │
    ├──► AutoDetectParser ──► Composite Detection
    │                              │
    │                              ▼
    │                         Select Parser
    │                              │
    ▼                              ▼
Parse Content ◄──────────── Specific Parser
    │                              │
    ▼                              ▼
ContentHandler ◄──────────── Extract Content
    │                              │
    ▼                              ▼
Metadata    ◄──────────────── Extract Metadata
```

## Data Flow

1. **Input**: Document bytes + Metadata object
2. **Detection**: Identify MIME type
3. **Parser Selection**: Choose appropriate parser
4. **Parsing**: Extract content and metadata
5. **Output**: Text content + populated Metadata

## Module Dependencies

```
tika-app ──────┐
               ├──► tika-core ◄──── All modules depend on core
tika-server ───┤         │
               │         └──► tika-parsers-standard
tika-grpc ─────┘                    │
                                    ├──► tika-parsers-extended
tika-pipes ─────────────────────────┘
                                    
tika-eval ──────────► (independent evaluation tools)
```

## Extension Points

1. **Custom Parsers**: Implement `Parser` interface
2. **Custom Detectors**: Implement `Detector` interface
3. **Custom Handlers**: Extend `ContentHandlerDecorator`
4. **Custom Fetchers**: Implement `Fetcher` interface (pipes)
5. **Custom Emitters**: Implement `Emitter` interface (pipes)

## Security Architecture

- **Resource Limits**: Memory, time, recursion depth limits
- **Sandboxing**: Optional separate process parsing
- **Input Validation**: Defensive parsing practices
- **Dependency Management**: Regular security updates

## Performance Considerations

- **Streaming**: SAX-based streaming parsing
- **Lazy Loading**: Load parsers only when needed
- **Pooling**: Reusable parser instances
- **Async Processing**: tika-pipes for batch jobs
- **Caching**: MIME type detection caching

## Related Documentation

- [Design Patterns](./patterns.md)
- [Core Components](../components/tika-core/overview.md)
- [Parser Architecture](../components/tika-parsers/overview.md)
- [Server Architecture](../components/tika-server/overview.md)
