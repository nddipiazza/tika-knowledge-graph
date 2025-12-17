# External Dependencies

## Overview

Apache Tika depends on numerous external libraries for parsing different document formats and providing core functionality. This document catalogs the major external dependencies.

## Core Dependencies

### Apache Commons
Multiple Apache Commons libraries provide fundamental utilities:

**commons-io**
- Version: ~2.15.0+
- Purpose: I/O utilities, file operations
- Usage: Throughout Tika for stream handling

**commons-codec**
- Version: ~1.16.0+
- Purpose: Encoding/decoding (Base64, Hex, etc.)
- Usage: Metadata encoding, content processing

**commons-compress**
- Version: ~1.25.0+
- Purpose: Archive and compression formats
- Usage: ZIP, TAR, GZIP, 7Z, RAR parsing

### Logging

**SLF4J (Simple Logging Facade for Java)**
- Version: ~2.0.0+
- Purpose: Logging abstraction
- Usage: All Tika modules use SLF4J

## Parser Dependencies

### Microsoft Office Formats

**Apache POI**
- Version: ~5.2.0+
- Modules:
  - `poi` - Core Office formats (DOC, XLS, PPT)
  - `poi-ooxml` - Modern Office (DOCX, XLSX, PPTX)
  - `poi-scratchpad` - Legacy format support
  - `poi-ooxml-lite` - Lightweight OOXML
- Purpose: Microsoft Office document parsing
- Usage: All Office format parsers

**Dependencies of POI**:
- Apache Commons Collections
- Apache Commons Math
- SparseBitSet
- Curvesapi

### PDF Processing

**Apache PDFBox**
- Version: ~3.0.0+
- Modules:
  - `pdfbox` - Core PDF processing
  - `fontbox` - Font handling
  - `jempbox` - XMP metadata
  - `xmpbox` - XMP utilities
- Purpose: PDF parsing and manipulation
- Usage: PDF parser, metadata extraction

**Bouncy Castle**
- Version: ~1.77+
- Modules:
  - `bcprov-jdk18on` - Cryptography provider
  - `bcmail-jdk18on` - S/MIME support
- Purpose: Encryption/decryption for PDFs
- Usage: Encrypted PDF handling

### HTML/XML Processing

**TagSoup**
- Version: ~1.2+
- Purpose: Lenient HTML parsing
- Usage: HTML parser for malformed HTML

**Xerces**
- Version: Built into JDK
- Purpose: XML parsing
- Usage: XML document parsing

### Email Formats

**Apache James Mime4j**
- Version: ~0.8.9+
- Purpose: MIME message parsing
- Usage: Email (RFC822, EML) parsing

**Apache POI (for PST)**
- Version: Same as Office
- Purpose: Outlook PST file parsing

### Image Processing

**Java Advanced Imaging (JAI)**
- Version: Optional
- Purpose: Advanced image operations
- Usage: TIFF and advanced image format support

**Apache Commons Imaging**
- Version: ~1.0-alpha3+
- Purpose: Image metadata extraction
- Usage: EXIF, IPTC metadata from images

**TwelveMonkeys ImageIO**
- Version: ~3.10.0+
- Purpose: Extended ImageIO support
- Usage: Additional image format support

### Optical Character Recognition (OCR)

**Tesseract OCR**
- Version: 4.x or 5.x (native binary)
- Purpose: OCR text extraction from images/PDFs
- Usage: Optional OCR functionality
- Note: Requires native installation, not a Java dependency

### Audio/Video Metadata

**JAudioTagger**
- Version: ~2.2.5+
- Purpose: Audio metadata extraction
- Usage: MP3, FLAC, OGG metadata

**XuggleXuggler** (deprecated)
- Replaced by: Tika internal implementations
- Purpose: Video metadata

### Scientific Formats

**NetCDF-Java**
- Version: ~5.5.0+
- Purpose: NetCDF, HDF, GRIB parsing
- Usage: Scientific data formats

**JHDF**
- Version: ~0.6.0+
- Purpose: HDF5 file parsing
- Usage: Scientific data analysis

### Archive Formats

**Apache Commons Compress**
- Already listed in Core Dependencies
- Supports: ZIP, TAR, GZIP, BZIP2, XZ, 7Z, AR, CPIO, etc.

**Junrar**
- Version: ~7.5.0+
- Purpose: RAR archive extraction
- Usage: RAR file parser

### Language Detection

**Apache Tika OptimaizeLangDetect**
- Version: Internal to tika-langdetect
- Purpose: Language identification
- Usage: Language detection module

**CLD2** (Compact Language Detector)
- Optional native dependency
- Better accuracy than Java implementations

### Database Formats

**SQLite JDBC**
- Version: ~3.44.0+
- Purpose: SQLite database reading
- Usage: SQLite parser

**Jackcess**
- Version: ~4.0.0+
- Purpose: Microsoft Access (MDB/ACCDB) parsing
- Usage: Access database parser

### Geospatial

**Apache SIS**
- Version: ~1.4+
- Purpose: Geospatial metadata
- Usage: Geographic metadata extraction

**GeoAPI**
- Version: ~3.0.2+
- Purpose: Geospatial interfaces
- Usage: Coordinate systems, projections

### Text Encoding Detection

**ICU4J**
- Version: ~74.0+
- Purpose: International Components for Unicode
- Usage: Character encoding detection

**juniversalchardet**
- Version: ~2.4.0+
- Purpose: Charset detection (Mozilla algorithm)
- Usage: Encoding detection fallback

### CAD Formats

**DWG libraries**
- Various third-party libraries
- Purpose: AutoCAD DWG parsing
- Note: Limited support due to proprietary format

### Machine Learning

**DL4J (DeepLearning4J)**
- Version: Optional
- Purpose: Deep learning models
- Usage: Advanced content analysis

**ONNX Runtime**
- Version: Optional
- Purpose: Neural network inference
- Usage: ML-based parsing

## Build Dependencies

### Maven Plugins

**maven-compiler-plugin**
- Version: ~3.11.0+
- Purpose: Java compilation

**maven-surefire-plugin**
- Version: ~3.0.0+
- Purpose: Unit test execution

**maven-jar-plugin**
- Version: ~3.3.0+
- Purpose: JAR creation

**maven-assembly-plugin**
- Version: ~3.6.0+
- Purpose: Distribution packaging

**maven-shade-plugin**
- Version: ~3.5.0+
- Purpose: Uber JAR creation (tika-app)

**jacoco-maven-plugin**
- Version: ~0.8.11+
- Purpose: Code coverage

**ossindex-maven-plugin**
- Version: ~3.2.0+
- Purpose: Dependency vulnerability scanning

### Code Quality

**Checkstyle**
- Version: ~10.12.0+
- Purpose: Code style checking

**SpotBugs**
- Version: ~4.7.3+
- Purpose: Bug detection

**Error Prone**
- Version: ~2.23.0+
- Purpose: Static analysis

## Testing Dependencies

**JUnit**
- Version: ~5.10.0+ (Jupiter)
- Purpose: Unit testing framework

**Mockito**
- Version: ~5.7.0+
- Purpose: Mocking framework

**TestContainers**
- Version: ~1.19.0+
- Purpose: Docker-based integration tests
- Usage: tika-integration-tests

**XMLUnit**
- Version: ~2.9.0+
- Purpose: XML comparison in tests

## Server Dependencies

### JAX-RS / REST

**Apache CXF**
- Version: ~4.0.0+
- Purpose: JAX-RS implementation
- Usage: tika-server REST endpoints

**Jackson**
- Version: ~2.16.0+
- Purpose: JSON serialization
- Usage: JSON responses in tika-server

**Jersey** (alternative)
- Some server variants use Jersey instead of CXF

### gRPC

**gRPC Java**
- Version: ~1.59.0+
- Purpose: gRPC protocol
- Usage: tika-grpc module

**Protobuf**
- Version: ~3.25.0+
- Purpose: Protocol buffer serialization
- Usage: gRPC message definitions

## Dependency Management

### Bill of Materials (BOM)

Tika provides `tika-bom` to manage dependency versions:

```xml
<dependency>
  <groupId>org.apache.tika</groupId>
  <artifactId>tika-bom</artifactId>
  <version>4.0.0-SNAPSHOT</version>
  <type>pom</type>
  <scope>import</scope>
</dependency>
```

### Version Alignment

All Tika modules use consistent dependency versions defined in `tika-parent/pom.xml`.

## Security Considerations

### Vulnerability Scanning

- **ossindex-maven-plugin**: Scans for known vulnerabilities
- Regular dependency updates via Dependabot
- Security patches applied promptly

### Known Issues

Check SECURITY.md and Jira for security advisories.

## Optional Dependencies

Many dependencies are optional based on use case:
- **Tesseract**: Only if OCR is needed
- **ML libraries**: Only for ML-based parsers
- **Database drivers**: Only for DB format parsing
- **Native libraries**: Platform-specific

## Dependency Conflicts

### Resolution Strategies

1. Use `tika-bom` or `tika-parent` for version management
2. Exclude transitive dependencies if conflicts occur
3. Refer to compatibility matrix in documentation

## License Compatibility

All dependencies are Apache 2.0 compatible:
- Apache License 2.0
- MIT License
- BSD License
- Public Domain

**Note**: Some optional dependencies may have different licenses (check before use).

## Related Documentation

- [Internal Dependencies](./internal.md)
- [Version Matrix](./version-matrix.md)
- [Build Configuration](../configuration/build.md)

## Updating Dependencies

See developer documentation for dependency update procedures and testing requirements.
