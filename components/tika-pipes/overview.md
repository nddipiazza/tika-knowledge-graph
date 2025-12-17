# Tika Pipes Module

## Overview

**Module**: tika-pipes  
**Path**: `tika-pipes/`  
**Purpose**: Asynchronous batch document processing pipeline framework

Tika Pipes provides a scalable, asynchronous architecture for processing large volumes of documents. It separates concerns between fetching documents, parsing them, and emitting results, enabling parallel processing and fault tolerance.

## Architecture

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Fetcher  │────▶│  Parser  │────▶│ Emitter  │────▶│  Output  │
│ (Source) │     │ (Process)│     │ (Sink)   │     │(Storage) │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                 │                 │
     │           ┌─────▼─────┐           │
     │           │  Async    │           │
     │           │  Process  │           │
     │           │   Pool    │           │
     │           └───────────┘           │
     │                                   │
     └──────────── Pipes Config ─────────┘
```

## Core Concepts

### 1. Fetchers
Retrieve documents from various sources.

**Interface**: `org.apache.tika.pipes.fetcher.Fetcher`

**Available Fetchers**:
- **FileSystemFetcher**: Local file system
- **HttpFetcher**: HTTP/HTTPS URLs
- **S3Fetcher**: AWS S3 buckets
- **GCSFetcher**: Google Cloud Storage
- **AzureBlobFetcher**: Azure Blob Storage
- **MicrosoftGraphFetcher**: Microsoft Graph API
- **FSListFetcher**: File system listing/crawling

**Key Methods**:
```java
InputStream fetch(String fetchKey, Metadata metadata)
```

### 2. Parsers
Process documents using Tika's parsing infrastructure.

**Type**: `PipesParser` wraps standard Tika parsers

**Features**:
- Runs in isolated processes (fork mode)
- Timeout protection
- Resource limits
- Crash recovery

### 3. Emitters
Send parsed results to destinations.

**Interface**: `org.apache.tika.pipes.emitter.Emitter`

**Available Emitters**:
- **FileSystemEmitter**: Write to file system
- **S3Emitter**: Upload to AWS S3
- **OpenSearchEmitter**: Index to OpenSearch
- **SolrEmitter**: Index to Apache Solr
- **JDBCEmitter**: Write to database
- **KafkaEmitter**: Send to Kafka topics
- **JSONEmitter**: Output as JSON
- **CSVEmitter**: Output as CSV

**Key Methods**:
```java
void emit(String emitKey, List<Metadata> metadataList)
```

### 4. FetchEmitTuple
Core data structure linking fetch, parse, and emit operations.

**Fields**:
- `fetchKey`: Identifier for fetching
- `emitKey`: Identifier for emitting
- `metadata`: Metadata for the operation
- `onParseException`: Error handling strategy

## Module Structure

### tika-pipes-api
Core interfaces and abstract classes.

**Key Interfaces**:
- `Fetcher` - Document retrieval contract
- `Emitter` - Result output contract
- `FetchEmitTuple` - Processing unit

### tika-pipes-core
Core pipeline processing logic.

**Key Classes**:
- `PipesConfig` - Pipeline configuration
- `PipesClient` - Client-side processing
- `PipesServer` - Server-side coordination
- `AbstractComponentManager` - Component lifecycle
- `FetcherManager` - Fetcher management
- `EmitterManager` - Emitter management

**Async Processing**:
- `AsyncProcessor` - Asynchronous document processing
- `AsyncConfig` - Async configuration
- Thread pool management
- Queue management

### tika-pipes-fork-parser
Fork-based isolated parsing.

**Purpose**: Run parsers in separate processes for isolation and crash protection

**Features**:
- Process isolation
- Timeout handling
- Memory limit enforcement
- Automatic restart on crash

### tika-async-cli
Command-line interface for async processing.

**Usage**:
```bash
java -jar tika-async-cli.jar \
  -c pipes-config.xml \
  -numWorkers 10 \
  input-directory
```

### tika-pipes-iterator-commons
Common iterators for document sources.

**Iterators**:
- File system crawlers
- Directory watchers
- Custom iterators

### tika-pipes-reporter-commons
Progress and status reporting.

**Reporters**:
- Console reporter
- File-based reporter
- Database reporter
- Metric reporters

### tika-pipes-plugins
Fetcher and emitter plugin implementations.

**Plugin Modules**:
- `tika-pipes-s3` - AWS S3
- `tika-pipes-gcs` - Google Cloud Storage
- `tika-pipes-azure-blob` - Azure Blob
- `tika-pipes-http` - HTTP/HTTPS
- `tika-pipes-fs` - File System
- `tika-pipes-opensearch` - OpenSearch
- `tika-pipes-solr` - Apache Solr
- `tika-pipes-jdbc` - JDBC/Database
- `tika-pipes-kafka` - Apache Kafka
- `tika-pipes-msgraph` - Microsoft Graph
- `tika-pipes-json` - JSON output
- `tika-pipes-csv` - CSV output

## Configuration

### XML Configuration Example

```xml
<?xml version="1.0" encoding="UTF-8"?>
<properties>
  <fetchers>
    <fetcher class="org.apache.tika.pipes.fetcher.fs.FileSystemFetcher">
      <params>
        <param name="name" type="string">fsf</param>
        <param name="basePath" type="string">/data/input</param>
      </params>
    </fetcher>
  </fetchers>
  
  <emitters>
    <emitter class="org.apache.tika.pipes.emitter.opensearch.OpenSearchEmitter">
      <params>
        <param name="name" type="string">ose</param>
        <param name="url" type="string">http://localhost:9200</param>
        <param name="index" type="string">documents</param>
      </params>
    </emitter>
  </emitters>
  
  <pipes>
    <params>
      <param name="tikaConfig" type="string">tika-config.xml</param>
      <param name="numClients" type="int">10</param>
      <param name="maxForEmitBatchBytes" type="long">10000000</param>
      <param name="emitWithinMillis" type="long">60000</param>
    </params>
  </pipes>
</properties>
```

### JSON Configuration Example

```json
{
  "fetchers": [
    {
      "name": "s3f",
      "class": "org.apache.tika.pipes.fetcher.s3.S3Fetcher",
      "params": {
        "region": "us-east-1",
        "bucket": "my-documents"
      }
    }
  ],
  "emitters": [
    {
      "name": "solr",
      "class": "org.apache.tika.pipes.emitter.solr.SolrEmitter",
      "params": {
        "solrUrl": "http://localhost:8983/solr",
        "collection": "docs"
      }
    }
  ],
  "pipes": {
    "numClients": 10,
    "timeoutMillis": 300000
  }
}
```

## Processing Modes

### 1. Async Mode (Default)
Multiple worker threads process documents in parallel.

**Configuration**:
- `numClients`: Number of worker threads
- `maxForEmitBatchBytes`: Batch size for emitting
- `emitWithinMillis`: Max time before forcing emit

### 2. Fork Mode
Each document parsed in isolated process.

**Benefits**:
- Crash isolation
- Memory leak protection
- Resource limit enforcement

**Configuration**:
- `javaPath`: Path to Java executable
- `maxFiles`: Max files per forked process
- `timeoutMillis`: Process timeout

### 3. Server Mode
Persistent server accepting processing requests.

**Use Case**: Long-running service processing

## Error Handling

### Parse Exception Strategies
- `SKIP`: Skip failed documents
- `EMIT`: Emit with error metadata
- `TERMINATE`: Stop processing

### Retry Logic
- Configurable retry attempts
- Exponential backoff
- Dead letter queue support

## Monitoring & Metrics

### Built-in Metrics
- Documents processed
- Success/failure counts
- Processing time
- Queue depth
- Thread pool status

### Custom Reporters
Implement `PipesReporter` interface for custom monitoring.

## Performance Tuning

### Thread Pool Configuration
```xml
<param name="numClients" type="int">10</param>
<param name="queueSize" type="int">1000</param>
```

### Batching
```xml
<param name="maxForEmitBatchBytes" type="long">10000000</param>
<param name="maxForEmitBatch" type="int">100</param>
```

### Memory Management
```xml
<param name="maxFileSizeBytes" type="long">1000000000</param>
<param name="maxEmbeddedResources" type="int">1000</param>
```

## Use Cases

### 1. Batch Document Processing
Process thousands of documents from S3, emit to OpenSearch:
```bash
java -jar tika-async-cli.jar \
  -c s3-to-opensearch.xml \
  -numWorkers 20 \
  s3://bucket/path
```

### 2. File System Monitoring
Watch directory for new files and process automatically.

### 3. ETL Pipeline
Extract, transform, load documents from various sources.

### 4. Search Index Building
Crawl documents and build search indexes.

### 5. Document Migration
Migrate documents from one storage to another with parsing.

## Component Management (New in 4.0)

### Dynamic Configuration
Runtime addition/modification of fetchers and emitters:

**Add Fetcher**:
```java
FetcherManager manager = FetcherManager.load(...);
ExtensionConfig config = new ExtensionConfig(
    "myFetcher", 
    "file-system-fetcher", 
    "{\"basePath\": \"/data\"}"
);
manager.saveFetcher(config);
```

**Update Emitter**:
```java
EmitterManager manager = EmitterManager.load(...);
ExtensionConfig config = new ExtensionConfig(
    "myEmitter",
    "solr-emitter",
    "{\"solrUrl\": \"http://new-host:8983/solr\"}"
);
manager.saveEmitter(config);  // Updates existing, clears cache
```

## Integration with Other Modules

### With tika-server
Pipes can be used server-side for async processing.

### With tika-grpc
See [tika-grpc documentation](../tika-grpc/overview.md) for gRPC-based pipes control.

### With tika-eval
Evaluation and quality metrics for processed documents.

## Testing

### Integration Tests
Module: `tika-pipes-integration-tests`

Tests:
- Fetcher/Emitter functionality
- Async processing
- Error handling
- Component management

### Test Utilities
- Mock fetchers/emitters
- Test containers for external services
- Performance benchmarks

## Related Documentation

- [Fetchers](./tika-pipes/fetchers.md)
- [Emitters](./tika-pipes/emitters.md)
- [Configuration Guide](../../configuration/pipes-config.md)
- [gRPC Server](../tika-grpc/overview.md)
- [Performance Tuning](../../configuration/performance.md)

## Migration from 3.x

**Breaking Changes**:
- Component manager now supports updates (no exception on duplicate IDs)
- Async configuration structure changed
- New plugin system for fetchers/emitters

**Migration Steps**:
1. Update configuration XML/JSON format
2. Update error handling for new strategies
3. Review component manager usage for updates

---

**Source Code**: `tika-pipes/`
**Documentation**: [Apache Tika Pipes Guide](https://tika.apache.org/pipes.html)
