# Tika Parsers Module

## Overview

**Module**: tika-parsers  
**Path**: `/home/ndipiazza/source/github/apache/tika/tika-parsers`

The parsers module is a multi-module project containing implementations of the Parser interface for various document formats. It is split into standard and extended packages to separate common formats from specialized ones.

## Module Structure

```
tika-parsers/
├── tika-parsers-standard/
│   └── tika-parsers-standard-package/  # Common document formats
├── tika-parsers-extended/
│   ├── tika-parser-scientific-module/   # Scientific formats
│   ├── tika-parser-sqlite3-module/      # Database formats
│   ├── tika-parser-code-module/         # Source code
│   ├── tika-parser-pdf-module/          # PDF parsing
│   ├── tika-parser-image-module/        # Image formats
│   └── ...                              # Other specialized parsers
└── tika-parsers-ml/                     # Machine learning parsers
```

## Standard Parsers Package

### Supported Formats

#### Microsoft Office (Legacy)
- **DOC** - Word documents (via Apache POI)
- **XLS** - Excel spreadsheets (via Apache POI)
- **PPT** - PowerPoint presentations (via Apache POI)
- **Visio** - Visio diagrams
- **Publisher** - Publisher documents
- **Outlook MSG** - Email messages

#### Microsoft Office (OOXML)
- **DOCX** - Word documents
- **XLSX** - Excel spreadsheets
- **PPTX** - PowerPoint presentations

#### OpenDocument Format (ODF)
- **ODT** - Text documents
- **ODS** - Spreadsheets
- **ODP** - Presentations
- **ODG** - Graphics

#### PDF
- **PDF** - Portable Document Format (via Apache PDFBox)
- Supports text extraction, metadata, and OCR integration

#### Rich Text
- **RTF** - Rich Text Format

#### HTML/XML
- **HTML** - HyperText Markup Language
- **XHTML** - Extensible HTML
- **XML** - Extensible Markup Language

#### Text
- **TXT** - Plain text
- **CSV** - Comma-separated values
- **TSV** - Tab-separated values

#### Compression & Archives
- **ZIP** - ZIP archives
- **TAR** - TAR archives
- **GZIP** - GZIP compression
- **BZIP2** - BZIP2 compression
- **7Z** - 7-Zip archives
- **RAR** - RAR archives

#### Images
- **JPEG** - JPEG images
- **PNG** - PNG images
- **GIF** - GIF images
- **TIFF** - TIFF images
- **BMP** - Bitmap images
- **WebP** - WebP images

#### Email
- **EML** - Email messages (RFC 822)
- **MBOX** - Mailbox format
- **PST** - Outlook data files

#### E-books
- **EPUB** - Electronic publication
- **MOBI** - Mobipocket format

#### Audio
- **MP3** - MP3 audio (metadata only)
- **FLAC** - FLAC audio
- **Vorbis** - Ogg Vorbis

#### Video
- **MP4** - MPEG-4 video (metadata only)
- **AVI** - AVI video
- **FLV** - Flash video

## Extended Parsers

### Scientific Formats
- **NetCDF** - Network Common Data Form
- **HDF** - Hierarchical Data Format
- **GRIB** - Gridded Binary
- **FITS** - Flexible Image Transport System

### Database Formats
- **SQLite** - SQLite database files
- **DBF** - dBase database files

### CAD Formats
- **DWG** - AutoCAD drawing
- **DXF** - Drawing Exchange Format

### Geographic Formats
- **KML** - Keyhole Markup Language
- **GPX** - GPS Exchange Format

### Code/Programming
- **Java** - Java source code
- **C/C++** - C/C++ source code
- **Python** - Python source code
- **JavaScript** - JavaScript code

### Font Formats
- **TTF** - TrueType fonts
- **OTF** - OpenType fonts

## Key Parser Classes

### CompositeParser
Aggregates multiple parsers and delegates to the appropriate one based on MIME type.

**Location**: `org.apache.tika.parser.CompositeParser`

### AutoDetectParser
Automatically detects document type and selects appropriate parser.

**Location**: `org.apache.tika.parser.AutoDetectParser`

### Office Parsers
**Microsoft Office**: `org.apache.tika.parser.microsoft.*`
- `OfficeParser` - Legacy Office formats
- `OOXMLParser` - Modern Office formats

**OpenOffice**: `org.apache.tika.parser.odf.*`
- `OpenDocumentParser` - ODF formats

### PDF Parser
**Location**: `org.apache.tika.parser.pdf.PDFParser`

**Features**:
- Text extraction
- Metadata extraction
- Image extraction
- OCR integration (with Tesseract)
- Form data extraction
- Annotation extraction

### HTML Parser
**Location**: `org.apache.tika.parser.html.HtmlParser`

**Features**:
- HTML parsing with Tagsoup
- Boilerplate removal
- Encoding detection
- Metadata extraction from meta tags

### Email Parsers
**RFC822 Parser**: `org.apache.tika.parser.mail.RFC822Parser`
- Email message parsing
- Attachment extraction
- Header parsing

### Archive Parsers
**Location**: `org.apache.tika.parser.pkg.*`
- `ZipParser` - ZIP archives
- `TarParser` - TAR archives
- `CompressorParser` - Various compression formats

## Parser Dependencies

### Apache POI
Microsoft Office document parsing
- poi (core)
- poi-ooxml (OOXML formats)
- poi-scratchpad (legacy formats)

### Apache PDFBox
PDF document parsing and manipulation

### Tagsoup
HTML parsing and cleanup

### Apache Commons Compress
Archive and compression format support

### Tesseract OCR (Optional)
Optical Character Recognition
- Not included by default
- Requires native installation

### Bouncy Castle
Cryptography support for encrypted documents

## Embedded Resource Extraction

Parsers support recursive parsing of embedded documents:

**Examples**:
- Attachments in emails
- Embedded images in PDFs
- Files within ZIP archives
- Embedded objects in Office documents

**Handler**: `org.apache.tika.extractor.EmbeddedDocumentExtractor`

## Parser Configuration

### Resource Limits
Prevent resource exhaustion:
- Max string length
- Max embedded resources
- Max uncompress size
- Timeout limits

### OCR Configuration
Configure Tesseract OCR:
- Language selection
- DPI settings
- Image preprocessing
- Timeout limits

### PDF Configuration
PDF-specific settings:
- Extract inline images
- OCR strategy
- Extract actions
- Access permissions

## Testing

Each parser module includes:
- Unit tests with sample documents
- Integration tests
- Performance benchmarks

**Test Resources**: `src/test/resources/test-documents/`

## Performance Considerations

### Memory Management
- Streaming parsing when possible
- Temp file usage for large documents
- Resource pooling

### Lazy Loading
- Parsers loaded only when needed
- Service Provider Interface (SPI)

### Parallel Processing
- See [tika-pipes](../tika-pipes/overview.md) for async processing

## Custom Parser Development

### Steps to Add a Custom Parser

1. Implement `Parser` interface or extend `AbstractParser`
2. Define supported MIME types
3. Implement parse logic
4. Create service provider configuration
5. Add to META-INF/services/org.apache.tika.parser.Parser

### Example Structure
```java
public class CustomParser extends AbstractParser {
    @Override
    public Set<MediaType> getSupportedTypes(ParseContext context) {
        return Collections.singleton(MediaType.parse("application/custom"));
    }
    
    @Override
    public void parse(InputStream stream, ContentHandler handler,
                      Metadata metadata, ParseContext context) {
        // Implementation
    }
}
```

## Related Documentation

- [Core Module](../tika-core/overview.md)
- [Detector Module](../tika-detectors/overview.md)
- [Parser API](../../apis/parser-api.md)
- [Dependencies](../../dependencies/external.md)

## Source Code Locations

- Standard parsers: `tika-parsers/tika-parsers-standard/`
- Extended parsers: `tika-parsers/tika-parsers-extended/`
- ML parsers: `tika-parsers/tika-parsers-ml/`
