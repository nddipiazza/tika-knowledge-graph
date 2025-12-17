# Tika Server Module

## Overview

**Module**: tika-server  
**Path**: `/home/ndipiazza/source/github/apache/tika/tika-server`

Tika Server provides HTTP REST endpoints for all Tika functionality, allowing applications in any language to use Tika over HTTP. It's built on Apache CXF and runs as a standalone JAR.

## Server Variants

### tika-server-standard
Standard REST server with common parsers

### tika-server-classic  
Legacy server implementation

## Architecture

```
HTTP Requests
     │
     ▼
┌─────────────────┐
│   JAX-RS API    │
│  (CXF/Jersey)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   Resources     │
│  (Endpoints)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Tika Core API  │
└─────────────────┘
```

## Key Endpoints

### /tika (Metadata + Content)
Extract text and metadata from documents.

**Methods**:
- `PUT /tika` - Auto-detect and parse
- `PUT /tika/{mimetype}` - Parse with specified type

**Response**: Plain text content

### /meta (Metadata Only)
Extract only metadata without text content.

**Methods**:
- `PUT /meta` - Extract metadata
- `PUT /meta/{mimetype}` - With type hint

**Response**: JSON or CSV metadata

### /rmeta (Recursive Metadata)
Extract metadata recursively for embedded documents.

**Methods**:
- `PUT /rmeta` - Recursive extraction
- `PUT /rmeta/{mimetype}` - With type hint

**Response**: JSON array of metadata objects

### /detect/stream
Detect MIME type from content.

**Methods**:
- `PUT /detect/stream` - Detect from bytes

**Response**: MIME type string

### /detect/container
Detect container format.

**Methods**:
- `PUT /detect/container` - Detect container

### /language/stream
Detect language of text.

**Methods**:
- `PUT /language/stream` - Language detection

**Response**: Language code

### /translate/all/{translator}/{dest-language}
Translate content to another language.

**Methods**:
- `PUT /translate/all/{translator}/{dest}` - Translate content

### /unpack
Extract files from archives.

**Methods**:
- `PUT /unpack` - Unpack archive

**Response**: ZIP file with extracted content

### /version
Get Tika version information.

**Methods**:
- `GET /version` - Version info

## Request Headers

### Content-Type
Specify document MIME type (optional, will auto-detect if not provided)

### Accept
Specify desired response format:
- `text/plain` - Plain text
- `application/json` - JSON
- `text/csv` - CSV (for metadata)
- `text/html` - HTML

### X-Tika-OCR-Language
OCR language hint (e.g., "eng", "fra")

### X-Tika-PDFextractInlineImages
Extract inline PDF images (true/false)

## Configuration

### Server Options

**Port**: Default 9998
```bash
java -jar tika-server-standard.jar -p 9998
```

**Host**: Bind to specific address
```bash
java -jar tika-server-standard.jar -h localhost
```

**CORS**: Enable CORS
```bash
java -jar tika-server-standard.jar --cors "*"
```

**Config File**: Custom Tika configuration
```bash
java -jar tika-server-standard.jar -c tika-config.xml
```

### Resource Limits

Protect against resource exhaustion:
- Max file size
- Request timeout
- Max content length
- Task timeout

**Example tika-config.xml**:
```xml
<properties>
  <parsers>
    <parser class="org.apache.tika.parser.DefaultParser">
      <params>
        <param name="maxStringLength" type="int">10000000</param>
      </params>
    </parser>
  </parsers>
</properties>
```

## Security Features

### 1. Server-Side Request Forgery (SSRF) Protection
Prevents malicious file URLs

### 2. Digest Authentication
Optional basic authentication

### 3. Resource Limits
- Memory limits
- Time limits
- Recursion depth limits

### 4. Separate Process Parsing
Optional forking to isolated process

### 5. Input Validation
Validates content types and sizes

## Error Handling

### HTTP Status Codes

- **200 OK**: Successful parsing
- **204 No Content**: Empty document
- **415 Unsupported Media Type**: Unknown format
- **422 Unprocessable Entity**: Parsing error
- **500 Internal Server Error**: Server error
- **503 Service Unavailable**: Server overloaded

### Error Response Format
```json
{
  "error": "Error message",
  "stack": "Stack trace (if enabled)"
}
```

## Docker Deployment

### Official Docker Images
Available on Docker Hub:
```bash
docker pull apache/tika:latest
docker run -p 9998:9998 apache/tika
```

### Custom Dockerfile
```dockerfile
FROM apache/tika:latest
COPY tika-config.xml /tika-config.xml
CMD ["--config", "/tika-config.xml"]
```

## Client Libraries

### Python
```python
import requests
response = requests.put('http://localhost:9998/tika', 
                        data=open('document.pdf', 'rb'))
text = response.text
```

### Java
```java
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("http://localhost:9998/tika"))
    .PUT(HttpRequest.BodyPublishers.ofFile(Path.of("document.pdf")))
    .build();
HttpResponse<String> response = client.send(request, 
    HttpResponse.BodyHandlers.ofString());
```

### cURL
```bash
curl -T document.pdf http://localhost:9998/tika
curl -T document.pdf -H "Accept: application/json" \
     http://localhost:9998/meta
```

## Monitoring & Health Checks

### Health Endpoint
`GET /` - Returns simple status page

### Metrics
Integration with monitoring tools:
- JMX metrics
- Prometheus metrics (with configuration)

### Logging
Configurable logging via:
- log4j2.xml
- JUL configuration

## Performance Tuning

### Thread Pool
Configure worker threads:
```bash
java -jar tika-server-standard.jar --numWorkers 10
```

### Memory
JVM memory settings:
```bash
java -Xmx2g -Xms1g -jar tika-server-standard.jar
```

### Connection Pooling
Reuse parser instances to reduce overhead

### Async Processing
Consider [tika-pipes](../tika-pipes/overview.md) for batch processing

## Comparison with tika-app

| Feature | tika-server | tika-app |
|---------|-------------|----------|
| Interface | HTTP REST | Command line |
| Deployment | Web service | Standalone tool |
| Concurrency | Multi-threaded | Single process |
| Language Support | Any (HTTP) | Java only |
| Use Case | Production services | Scripts, testing |

## Related Documentation

- [Server API Specification](../../apis/server-api.md)
- [gRPC Server](../tika-grpc/overview.md)
- [Deployment Guide](../../configuration/deployment.md)
- [Security Configuration](../../configuration/security.md)

## Source Code Locations

- Main source: `tika-server/tika-server-standard/src/main/java/`
- Resources: `tika-server/tika-server-standard/src/main/resources/`
- Tests: `tika-server/tika-server-standard/src/test/`

## Examples

See `tika-example/` for client examples in various languages.
