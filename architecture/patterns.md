# Design Patterns in Apache Tika

## Overview

Apache Tika employs several well-established design patterns to achieve its goals of extensibility, maintainability, and ease of use.

## 1. Facade Pattern

**Implementation**: `org.apache.tika.Tika`

The Tika class provides a simplified interface to the complex Tika subsystem. Instead of dealing with parsers, detectors, handlers, and configurations separately, users can use simple methods.

**Example Methods:**
```java
String detect(File file)
String parseToString(File file)
Reader parse(InputStream stream)
```

**Benefits:**
- Hides complexity from casual users
- Provides sensible defaults
- Single entry point for common operations

## 2. Strategy Pattern

**Implementation**: Parser and Detector interfaces

Different parsing and detection algorithms are encapsulated as interchangeable strategies.

**Key Interfaces:**
- `org.apache.tika.parser.Parser`
- `org.apache.tika.detect.Detector`

**Usage:**
- Swap parsers at runtime based on document type
- Chain detectors for multi-level detection
- Add new parsers without modifying existing code

## 3. Composite Pattern

**Implementations:**
- `org.apache.tika.parser.CompositeParser`
- `org.apache.tika.detect.CompositeDetector`

Treats individual parsers and groups of parsers uniformly. The CompositeParser contains multiple parsers and delegates to the appropriate one.

**Benefits:**
- Unified interface for single and multiple parsers
- Hierarchical parser organization
- Simplified client code

## 4. Decorator Pattern

**Implementation**: ContentHandler decorators

Various handler decorators wrap base handlers to add functionality:
- `org.apache.tika.sax.BodyContentHandler` - Extracts body text only
- `org.apache.tika.sax.WriteOutContentHandler` - Limits output size
- `org.apache.tika.sax.TeeContentHandler` - Splits output to multiple handlers

**Benefits:**
- Add behavior dynamically
- Compose multiple decorators
- Keep base handlers simple

## 5. Service Provider Interface (SPI)

**Implementation**: Java ServiceLoader mechanism

Parsers and detectors are discovered automatically through META-INF/services files:
- `META-INF/services/org.apache.tika.parser.Parser`
- `META-INF/services/org.apache.tika.detect.Detector`

**Benefits:**
- Plugin architecture
- No code changes to add parsers
- Loose coupling

## 6. Template Method Pattern

**Implementation**: `org.apache.tika.parser.AbstractParser`

Defines the skeleton of the parsing algorithm, with subclasses overriding specific steps.

**Common Structure:**
```java
public abstract class AbstractParser implements Parser {
    @Override
    public void parse(...) {
        // Common setup
        parseInternal(...);  // Subclass implements
        // Common cleanup
    }
    
    protected abstract void parseInternal(...);
}
```

## 7. Factory Pattern

**Implementation**: Parser creation and configuration

`TikaConfig` acts as a factory for creating properly configured parser and detector instances.

**Usage:**
- Create parsers based on configuration
- Initialize with appropriate settings
- Manage parser lifecycle

## 8. Singleton Pattern

**Implementation**: Default configuration instances

Some default configurations use singleton-like patterns:
- Default MIME types database
- Default parser configuration

**Note**: Not strict singletons to allow customization

## 9. Chain of Responsibility

**Implementation**: Detector chaining

Multiple detectors can be chained, each attempting detection until one succeeds:

1. `TypeDetector` - Uses explicitly declared type
2. `ResourceNameDetector` - Uses filename extension  
3. `MagicDetector` - Examines file content
4. `DefaultDetector` - Fallback detector

## 10. Observer Pattern

**Implementation**: SAX ContentHandler

The SAX ContentHandler acts as an observer receiving parsing events:
- `startDocument()` / `endDocument()`
- `startElement()` / `endElement()`
- `characters()`

**Benefits:**
- Event-driven parsing
- Memory efficient for large documents
- Multiple observers can process same events

## 11. Builder Pattern

**Implementation**: Configuration builders

Complex configurations use builder pattern:
- `TikaConfig.Builder`
- Parser configuration builders

**Benefits:**
- Fluent API
- Step-by-step construction
- Immutable configuration objects

## 12. Adapter Pattern

**Implementation**: Parser wrappers

Adapters wrap third-party parsers to conform to Tika's Parser interface:
- `PDFParser` adapts Apache PDFBox
- `POIFSContainerDetector` adapts Apache POI
- `TesseractOCRParser` adapts Tesseract

**Benefits:**
- Integrate external libraries
- Consistent interface
- Isolate third-party dependencies

## Pattern Combinations

### Auto-Detection Flow

Combines multiple patterns:
1. **Facade**: Tika.parse() is called
2. **Strategy**: AutoDetectParser selected
3. **Chain of Responsibility**: Detectors try in sequence
4. **Factory**: TikaConfig creates appropriate parser
5. **Composite**: CompositeParser contains specific parsers
6. **Adapter**: Specific parser wraps external library

## Anti-Patterns Avoided

1. **God Object**: Avoided by separating concerns across modules
2. **Tight Coupling**: SPI allows loose coupling
3. **Hardcoded Dependencies**: Configuration-driven parser selection
4. **Inheritance Overuse**: Favor composition over inheritance

## Related Documentation

- [Architecture Overview](./overview.md)
- [Core API](../components/tika-core/api.md)
- [Parser Implementation Guide](../components/tika-parsers/overview.md)
