# Tika Core Module

## Overview

**Module**: tika-core  
**Artifact**: org.apache.tika:tika-core  
**Path**: `/home/ndipiazza/source/github/apache/tika/tika-core`

The core module provides the fundamental APIs, interfaces, and base implementations that all other Tika modules depend on. It contains no actual parser implementations, only the contracts and supporting infrastructure.

## Purpose

Tika-core defines the architecture and provides:
- Common interfaces for parsers, detectors, and handlers
- Metadata handling and standardized metadata keys
- Configuration management
- MIME type database
- Base SAX content handlers
- Utility classes

## Key Packages

### org.apache.tika
Main facade class providing simple API for common operations.

**Key Class**: `Tika.java`
- Simple document parsing
- MIME type detection
- Language detection
- One-line convenience methods

### org.apache.tika.parser
Parser interface and related infrastructure.

**Key Interfaces/Classes:**
- `Parser` - Main parser interface
- `AbstractParser` - Base class for parsers
- `CompositeParser` - Aggregates multiple parsers
- `AutoDetectParser` - Automatically selects appropriate parser
- `ParseContext` - Context object for parsing parameters

### org.apache.tika.detect
MIME type detection interfaces and implementations.

**Key Interfaces/Classes:**
- `Detector` - MIME type detection interface
- `DefaultDetector` - Default composite detector
- `MagicDetector` - Magic byte-based detection
- `ResourceNameDetector` - Extension-based detection
- `TypeDetector` - Uses explicit type hints

### org.apache.tika.metadata
Metadata extraction and management.

**Key Classes:**
- `Metadata` - Key-value metadata container
- `TikaCoreProperties` - Standard metadata keys
- `HttpHeaders` - HTTP header constants
- `Office` - Office document metadata keys
- `Message` - Email metadata keys

### org.apache.tika.mime
MIME type management and utilities.

**Key Classes:**
- `MimeTypes` - MIME type database
- `MimeType` - Represents a MIME type
- `MediaType` - Immutable media type representation

### org.apache.tika.sax
SAX content handler implementations.

**Key Classes:**
- `BodyContentHandler` - Extracts body text
- `WriteOutContentHandler` - Limits output size
- `TeeContentHandler` - Splits to multiple handlers
- `ContentHandlerDecorator` - Base decorator class
- `XHTMLContentHandler` - XHTML output handler

### org.apache.tika.config
Configuration management.

**Key Classes:**
- `TikaConfig` - Main configuration class
- `ServiceLoader` - SPI service loading
- `InitializableProblemHandler` - Error handling during initialization

### org.apache.tika.io
I/O utilities and abstractions.

**Key Classes:**
- `TikaInputStream` - Enhanced input stream with temp file support
- `TemporaryResources` - Temporary resource management
- `FilenameUtils` - Filename manipulation
- `IOUtils` - I/O utilities

### org.apache.tika.language
Language detection interfaces.

**Key Interfaces:**
- `LanguageIdentifier` - Language detection contract
- `ProfilingHandler` - Content profiling for language detection

### org.apache.tika.exception
Custom exceptions.

**Key Exceptions:**
- `TikaException` - Base Tika exception
- `EncryptedDocumentException` - Document is encrypted
- `AccessPermissionException` - Insufficient permissions
- `ZeroByteFileException` - Empty file encountered

### org.apache.tika.utils
Utility classes.

**Key Classes:**
- `StringUtils` - String manipulation
- `XMLReaderUtils` - XML reader utilities
- `SystemUtils` - System information

## Core Interfaces

### Parser Interface

```java
public interface Parser extends Serializable {
    Set<MediaType> getSupportedTypes(ParseContext context);
    
    void parse(InputStream stream, 
               ContentHandler handler,
               Metadata metadata, 
               ParseContext context) 
        throws IOException, SAXException, TikaException;
}
```

### Detector Interface

```java
public interface Detector extends Serializable {
    MediaType detect(InputStream input, Metadata metadata) 
        throws IOException;
}
```

## Dependencies

### External Dependencies
- **slf4j-api**: Logging facade
- **commons-io**: I/O utilities
- **commons-codec**: Encoding utilities

### No Parser Dependencies
Tika-core intentionally has no dependencies on actual parsing libraries (PDFBox, POI, etc.). This keeps the core lightweight.

## Configuration

### Default Configuration
Located in: `src/main/resources/`
- `tika-config.xml` - Default Tika configuration
- `tika-mimetypes.xml` - MIME type definitions
- `custom-mimetypes.xml` - Custom MIME types

### TikaConfig

Central configuration management:
```java
TikaConfig config = new TikaConfig();
Parser parser = config.getParser();
Detector detector = config.getDetector();
```

## Key Features

### 1. MIME Type Database
Comprehensive MIME type database with:
- Magic byte patterns
- File extensions
- Glob patterns
- MIME type inheritance

### 2. Metadata Standards
Standardized metadata keys for:
- Dublin Core (title, creator, subject, etc.)
- Office documents (page count, word count, etc.)
- Geographic data (latitude, longitude, etc.)
- Digital media (width, height, duration, etc.)

### 3. Content Handler Chain
SAX-based content handling with:
- Event streaming
- Handler decoration
- Output filtering
- Size limiting

### 4. Resource Management
Automatic cleanup of:
- Temporary files
- Input streams
- Extraction resources

## API Patterns

### Simple Parsing
```java
Tika tika = new Tika();
String text = tika.parseToString(file);
String mimeType = tika.detect(file);
```

### Advanced Parsing
```java
Parser parser = new AutoDetectParser();
ContentHandler handler = new BodyContentHandler();
Metadata metadata = new Metadata();
ParseContext context = new ParseContext();

parser.parse(inputStream, handler, metadata, context);
```

## Testing Support

Tika-core provides testing utilities:
- `TikaTest` - Base test class
- Sample documents in test resources
- Mock parsers and detectors

## Related Documentation

- [Parser Architecture](../tika-parsers/overview.md)
- [Detector Details](../tika-detectors/overview.md)
- [Server Integration](../tika-server/overview.md)
- [API Documentation](../../apis/core-api.md)

## Source Code Locations

- Main source: `tika-core/src/main/java/org/apache/tika/`
- Resources: `tika-core/src/main/resources/`
- Tests: `tika-core/src/test/java/org/apache/tika/`
