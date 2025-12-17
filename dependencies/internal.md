# Internal Dependencies

## Overview

Apache Tika is organized as a multi-module Maven project. Understanding the internal dependencies between modules is crucial for development and customization.

## Module Dependency Hierarchy

### Foundation Layer

**tika-parent**
- Purpose: Parent POM with shared configuration
- Dependencies: None (top-level)
- Dependents: All modules

**tika-bom** (Bill of Materials)
- Purpose: Dependency version management
- Dependencies: None
- Dependents: External projects using Tika

**tika-core**
- Purpose: Core APIs and interfaces
- Dependencies: Minimal external (SLF4J, Commons IO)
- Dependents: All other Tika modules

### Detection Layer

**tika-detectors**
- Parent module for detector implementations
- Sub-modules:
  - `tika-detector-siegfried` - Siegfried-based detection
  - `tika-detector-tensorflow` - TensorFlow detection

### Parser Layer

**tika-parsers**
- Parent module for all parsers
- Sub-modules structure:

#### tika-parsers-standard
- Purpose: Common document format parsers
- Dependencies: tika-core, POI, PDFBox, Commons Compress
- Packages all standard parsers together

**tika-parsers-standard-package**
- Assembly module
- Includes: Office, PDF, HTML, Archives, Email, etc.

#### tika-parsers-extended
Individual parser modules for specialized formats:

**tika-parser-pdf-module**
- Dependencies: tika-core, PDFBox, Bouncy Castle
- Purpose: PDF parsing

**tika-parser-microsoft-module**
- Dependencies: tika-core, Apache POI
- Purpose: Microsoft Office formats

**tika-parser-image-module**
- Dependencies: tika-core, Commons Imaging, ImageIO
- Purpose: Image format parsing and metadata

**tika-parser-scientific-module**
- Dependencies: tika-core, NetCDF-Java, JHDF
- Purpose: Scientific data formats (NetCDF, HDF, GRIB)

**tika-parser-sqlite3-module**
- Dependencies: tika-core, SQLite JDBC, Jackcess
- Purpose: Database format parsing

**tika-parser-code-module**
- Dependencies: tika-core
- Purpose: Source code parsing

**tika-parser-miscoffice-module**
- Dependencies: tika-core, POI
- Purpose: Visio, Publisher, Outlook

**tika-parser-html-module**
- Dependencies: tika-core, TagSoup
- Purpose: HTML parsing

**tika-parser-xml-module**
- Dependencies: tika-core
- Purpose: XML parsing

**tika-parser-text-module**
- Dependencies: tika-core
- Purpose: Plain text and CSV

**tika-parser-pkg-module**
- Dependencies: tika-core, Commons Compress, Junrar
- Purpose: Archive and package formats

**tika-parser-mail-module**
- Dependencies: tika-core, Mime4j, POI
- Purpose: Email formats

**tika-parser-audiovideo-module**
- Dependencies: tika-core, JAudioTagger
- Purpose: Audio/video metadata

**tika-parser-font-module**
- Dependencies: tika-core, PDFBox
- Purpose: Font file parsing

**tika-parser-crypto-module**
- Dependencies: tika-core, Bouncy Castle
- Purpose: Encrypted content handling

#### tika-parsers-ml
**tika-parser-nlp-module**
- Dependencies: tika-core, OpenNLP
- Purpose: Natural language processing

**tika-parser-advancedmedia-module**
- Dependencies: tika-core, DL4J (optional)
- Purpose: Advanced media analysis

### Support Modules

**tika-xmp**
- Purpose: XMP metadata handling
- Dependencies: tika-core, XMP libraries
- Used by: PDF parser, image parsers

**tika-langdetect**
- Purpose: Language detection
- Dependencies: tika-core, language detection libraries
- Used by: Parsers needing language identification

**tika-translate**
- Purpose: Content translation
- Dependencies: tika-core, translation APIs
- Used by: tika-server translation endpoints

**tika-serialization**
- Purpose: Serialization formats (JSON, XML, CSV)
- Dependencies: tika-core, Jackson
- Used by: tika-server, tika-pipes

**tika-handlers**
- Purpose: Content handler implementations
- Dependencies: tika-core
- Used by: Custom content processing

### Application Modules

**tika-app**
- Purpose: Standalone command-line application
- Dependencies: tika-core, tika-parsers-standard-package, all detectors
- Packaging: Uber JAR with all dependencies
- Output: Single executable JAR

**tika-server**
Sub-modules:
- **tika-server-core**: Core server functionality
- **tika-server-standard**: Standard REST server
- **tika-server-classic**: Legacy server

Dependencies: tika-core, parsers, CXF, Jackson

**tika-grpc**
- Purpose: gRPC server implementation
- Dependencies: tika-core, parsers, gRPC, Protobuf

### Pipeline Modules

**tika-pipes**
Sub-modules:
- **tika-pipes-core**: Core pipeline framework
- **tika-pipes-reporters**: Reporting implementations
- **tika-pipes-fetchers**: Document fetchers
- **tika-pipes-emitters**: Result emitters

**tika-fetchers**
- **tika-fetcher-http**: HTTP document fetching
- **tika-fetcher-s3**: AWS S3 fetching
- **tika-fetcher-gcs**: Google Cloud Storage
- **tika-fetcher-az**: Azure Blob Storage
- **tika-fetcher-fs**: File system fetching

**tika-emitters**
- **tika-emitter-fs**: File system output
- **tika-emitter-s3**: AWS S3 output
- **tika-emitter-jdbc**: Database output
- **tika-emitter-opensearch**: OpenSearch indexing
- **tika-emitter-solr**: Solr indexing

### Utility Modules

**tika-eval**
- Purpose: Parser evaluation and benchmarking
- Dependencies: tika-core, parsers, H2 database
- Use: Quality assurance, performance testing

**tika-example**
- Purpose: Example code and usage demonstrations
- Dependencies: tika-core, tika-parsers
- Use: Learning, reference implementations

**tika-java7**
- Purpose: Legacy Java 7 compatibility (deprecated)
- Status: End of life

**tika-annotation-processor**
- Purpose: Annotation processing for Tika plugins
- Dependencies: Java annotation processing API
- Use: Compile-time plugin validation

**tika-plugins-core**
- Purpose: Plugin system core
- Dependencies: tika-core
- Use: Custom plugin development

### Testing Modules

**tika-integration-tests**
- Purpose: Integration and system tests
- Dependencies: All Tika modules, TestContainers
- Requires: Docker for containerized tests

**tika-bundles**
- Purpose: OSGi bundle packaging
- Dependencies: Various Tika modules
- Use: OSGi container deployment

## Dependency Graph

```
tika-core
   ├── tika-parsers-standard
   │   └── tika-app
   ├── tika-parsers-extended
   │   └── tika-app (optional)
   ├── tika-detectors
   │   └── tika-app, tika-server
   ├── tika-server
   │   ├── tika-server-standard
   │   └── tika-server-classic
   ├── tika-grpc
   ├── tika-pipes
   │   ├── tika-fetchers
   │   └── tika-emitters
   ├── tika-langdetect
   ├── tika-translate
   ├── tika-serialization
   └── tika-eval
```

## Circular Dependency Avoidance

Tika carefully avoids circular dependencies:
- Core never depends on parsers
- Parsers never depend on applications
- Server/App depend on core and parsers, not vice versa

## Module Selection Strategy

### Minimal Setup
- `tika-core` only - For custom parser development

### Standard Usage
- `tika-parsers-standard-package` - Common formats

### Comprehensive
- `tika-app` - Everything in one JAR

### Server Deployment
- `tika-server-standard` - REST API with standard parsers

### Batch Processing
- `tika-pipes` + fetchers + emitters - Async pipelines

## Version Compatibility

All modules within a Tika release share the same version number:
- Version: 4.0.0-SNAPSHOT
- All modules: 4.0.0-SNAPSHOT

### Cross-Version Compatibility

Generally, modules from different Tika versions should not be mixed:
- ✅ All modules from Tika 4.0.0
- ❌ tika-core 4.0.0 + tika-parsers 3.0.0

## Maven Module Declaration

In parent `pom.xml`:
```xml
<modules>
  <module>tika-parent</module>
  <module>tika-core</module>
  <module>tika-parsers</module>
  <module>tika-server</module>
  <module>tika-app</module>
  <!-- ... more modules -->
</modules>
```

## Build Order

Maven Reactor determines build order based on dependencies:
1. tika-parent
2. tika-bom
3. tika-core
4. tika-annotation-processor
5. tika-serialization
6. tika-detectors
7. tika-parsers (all sub-modules)
8. tika-xmp, tika-langdetect, tika-translate
9. tika-handlers
10. tika-pipes
11. tika-server
12. tika-grpc
13. tika-app
14. tika-eval
15. tika-integration-tests

## Development Dependencies

For development, you may need:
```xml
<!-- Build specific module -->
<dependency>
  <groupId>org.apache.tika</groupId>
  <artifactId>tika-core</artifactId>
  <version>${tika.version}</version>
</dependency>
```

With Maven:
```bash
# Build specific module and dependencies
mvn clean install -am -pl :tika-server-standard
```

## Related Documentation

- [External Dependencies](./external.md)
- [Build Configuration](../configuration/build.md)
- [Module Architecture](../architecture/overview.md)
