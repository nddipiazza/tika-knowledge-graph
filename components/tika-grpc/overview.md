# Tika gRPC Server Module

## Overview

**Module**: tika-grpc  
**Path**: `tika-grpc/`  
**Purpose**: gRPC-based server for Tika Pipes document processing

The Tika gRPC server provides a high-performance, language-agnostic API for document processing using the Tika Pipes framework. It manages a pool of Tika Pipes clients and exposes CRUD operations for fetchers plus document fetch/parse operations.

## Key Features

- **Multi-language Support**: gRPC works with any language (Java, Python, Go, C++, etc.)
- **Streaming Support**: Unary, server-side streaming, and bi-directional streaming
- **Dynamic Configuration**: Runtime fetcher CRUD operations
- **Connection Pooling**: Manages pool of Tika Pipes clients
- **Mutual TLS**: Built-in security via mTLS
- **High Performance**: Binary protocol with efficient serialization

## Architecture

```
┌─────────────────────────────────────────────┐
│         gRPC Clients (Any Language)         │
│  (Python, Go, Java, JavaScript, etc.)       │
└──────────────────┬──────────────────────────┘
                   │ gRPC Protocol (HTTP/2)
                   │ Protocol Buffers
                   ▼
┌─────────────────────────────────────────────┐
│          TikaGrpcServerImpl                 │
│  - Request handling                         │
│  - Component management                     │
│  - Stream coordination                      │
└──────────────┬──────────────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│Fetcher │ │Fetcher │ │Fetcher │
│Manager │ │Manager │ │Manager │
└───┬────┘ └───┬────┘ └───┬────┘
    │          │          │
    └──────────┼──────────┘
               ▼
    ┌─────────────────────┐
    │   Tika Pipes Pool   │
    │  (Async Processors) │
    └─────────────────────┘
```

## Protocol Definition

### Service: `Tika`

Defined in: `src/main/proto/tika.proto`

#### RPC Methods

##### 1. SaveFetcher (Unary)
Save or update a fetcher configuration.

**Request**: `SaveFetcherRequest`
- `fetcher_id`: Unique identifier
- `fetcher_class`: Full Java class name
- `fetcher_config_json`: JSON configuration

**Response**: `SaveFetcherReply`
- `fetcher_id`: Confirmed ID

**Example**:
```protobuf
SaveFetcherRequest {
  fetcher_id: "s3-docs"
  fetcher_class: "org.apache.tika.pipes.fetcher.s3.S3Fetcher"
  fetcher_config_json: "{\"region\":\"us-east-1\",\"bucket\":\"docs\"}"
}
```

##### 2. GetFetcher (Unary)
Retrieve fetcher configuration.

**Request**: `GetFetcherRequest`
- `fetcher_id`: Fetcher to retrieve

**Response**: `GetFetcherReply`
- `fetcher_id`: Fetcher ID
- `fetcher_class`: Class name
- `fetcher_config_json`: Configuration JSON

##### 3. ListFetchers (Unary)
List all available fetchers.

**Request**: `ListFetchersRequest` (empty)

**Response**: `ListFetchersReply`
- `fetcher_ids`: List of fetcher IDs

##### 4. DeleteFetcher (Unary)
Remove a fetcher from the store.

**Request**: `DeleteFetcherRequest`
- `fetcher_id`: Fetcher to delete

**Response**: `DeleteFetcherReply`
- `deleted`: Boolean success flag

##### 5. FetchAndParse (Unary)
Fetch and parse a single document, return result.

**Request**: `FetchAndParseRequest`
- `fetcher_id`: Fetcher to use
- `fetch_key`: Document identifier
- `additional_fetch_config_json`: Optional override config

**Response**: `FetchAndParseReply`
- `fetch_key`: Echo of request
- `fields`: Map of metadata fields
- `status`: Parse status (SUCCESS, TIMEOUT, ERROR, etc.)
- `error_message`: Error details if failed

##### 6. FetchAndParseServerSideStreaming (Server Streaming)
Fetch and parse, stream results as they complete.

**Use Case**: Processing multiple embedded documents or large batches

**Request**: Single `FetchAndParseRequest`

**Response**: Stream of `FetchAndParseReply` messages

##### 7. FetchAndParseBiDirectionalStreaming (Bi-directional Streaming)
Continuous fetch/parse pipeline with streaming in both directions.

**Use Case**: High-throughput batch processing

**Request**: Stream of `FetchAndParseRequest` messages

**Response**: Stream of `FetchAndParseReply` messages

##### 8. GetFetcherConfigJsonSchema (Unary)
Get JSON schema for fetcher configuration.

**Request**: `GetFetcherConfigJsonSchemaRequest`
- `fetcher_class`: Class name

**Response**: `GetFetcherConfigJsonSchemaReply`
- `json_schema`: JSON schema definition

## Implementation

### Core Classes

#### TikaGrpcServer
Main server class, handles startup and lifecycle.

**Location**: `org.apache.tika.pipes.grpc.TikaGrpcServer`

**Key Methods**:
```java
void start() throws IOException
void stop() throws InterruptedException
void blockUntilShutdown() throws InterruptedException
```

**Configuration**:
```java
TikaGrpcServer server = new TikaGrpcServer(
    port,
    tikaConfigPath,
    numClients,
    maxMessageSize
);
```

#### TikaGrpcServerImpl
Service implementation handling gRPC requests.

**Location**: `org.apache.tika.pipes.grpc.TikaGrpcServerImpl`

**Key Features**:
- FetcherManager integration
- Tika Pipes pool management
- Request/response marshalling
- Error handling

### Component Management

#### Runtime Configuration (New in 4.0)
The gRPC server supports dynamic fetcher management:

**Before (3.x)**: Used reflection hack to update fetchers
```java
// Old approach - reflection to clear cache
Field cacheField = AbstractComponentManager.class.getDeclaredField("cache");
cacheField.setAccessible(true);
Map<String, ?> cache = (Map<String, ?>) cacheField.get(fetcherManager);
cache.clear();
```

**After (4.0)**: Clean API for updates
```java
// New approach - saveComponent handles updates
fetcherManager.saveFetcher(config);  // Updates existing, clears cache
```

**Implementation Details**:
- `saveComponent()` checks if component exists
- If exists, clears cache and updates config
- If new, adds to component store
- Logging distinguishes create vs update

## Security

### Mutual TLS (mTLS)
Server supports mutual TLS for authentication.

**Configuration**:
```java
SslContext sslContext = GrpcSslContexts.forServer(
    certChainFile,
    privateKeyFile,
    trustCertCollectionFile
).clientAuth(ClientAuth.REQUIRE)
 .build();

server = NettyServerBuilder.forPort(port)
    .sslContext(sslContext)
    .build();
```

**Certificate Requirements**:
- Server certificate and private key
- Client certificate (for verification)
- Trust store with CA certificates

### Access Control
- Only authorized clients with valid certificates can connect
- Fetcher CRUD operations require authentication
- In-memory only configuration (no persistence by default)

## Client Examples

### Python Client

```python
import grpc
from tika_pb2 import SaveFetcherRequest, FetchAndParseRequest
from tika_pb2_grpc import TikaStub

# Connect to server
channel = grpc.insecure_channel('localhost:50051')
stub = TikaStub(channel)

# Save a fetcher
save_request = SaveFetcherRequest(
    fetcher_id='fs-fetcher',
    fetcher_class='org.apache.tika.pipes.fetcher.fs.FileSystemFetcher',
    fetcher_config_json='{"basePath": "/data"}'
)
save_response = stub.SaveFetcher(save_request)

# Fetch and parse
parse_request = FetchAndParseRequest(
    fetcher_id='fs-fetcher',
    fetch_key='document.pdf'
)
parse_response = stub.FetchAndParse(parse_request)

print(f"Status: {parse_response.status}")
print(f"Fields: {parse_response.fields}")
```

### Go Client

```go
import (
    "context"
    pb "path/to/tika"
    "google.golang.org/grpc"
)

conn, _ := grpc.Dial("localhost:50051", grpc.WithInsecure())
defer conn.Close()
client := pb.NewTikaClient(conn)

// Save fetcher
saveReq := &pb.SaveFetcherRequest{
    FetcherId: "fs-fetcher",
    FetcherClass: "org.apache.tika.pipes.fetcher.fs.FileSystemFetcher",
    FetcherConfigJson: `{"basePath": "/data"}`,
}
saveResp, _ := client.SaveFetcher(context.Background(), saveReq)

// Fetch and parse
parseReq := &pb.FetchAndParseRequest{
    FetcherId: "fs-fetcher",
    FetchKey: "document.pdf",
}
parseResp, _ := client.FetchAndParse(context.Background(), parseReq)
```

### Java Client

```java
ManagedChannel channel = ManagedChannelBuilder
    .forAddress("localhost", 50051)
    .usePlaintext()
    .build();

TikaGrpc.TikaBlockingStub stub = TikaGrpc.newBlockingStub(channel);

// Save fetcher
SaveFetcherRequest saveRequest = SaveFetcherRequest.newBuilder()
    .setFetcherId("fs-fetcher")
    .setFetcherClass("org.apache.tika.pipes.fetcher.fs.FileSystemFetcher")
    .setFetcherConfigJson("{\"basePath\": \"/data\"}")
    .build();
SaveFetcherReply saveReply = stub.saveFetcher(saveRequest);

// Fetch and parse
FetchAndParseRequest parseRequest = FetchAndParseRequest.newBuilder()
    .setFetcherId("fs-fetcher")
    .setFetchKey("document.pdf")
    .build();
FetchAndParseReply parseReply = stub.fetchAndParse(parseRequest);
```

## Streaming Examples

### Server-Side Streaming (Python)

```python
# Fetch and parse, receive stream of results
parse_request = FetchAndParseRequest(
    fetcher_id='fs-fetcher',
    fetch_key='archive.zip'  # Contains multiple documents
)

for response in stub.FetchAndParseServerSideStreaming(parse_request):
    print(f"Received: {response.fetch_key} - {response.status}")
```

### Bi-directional Streaming (Python)

```python
def request_generator():
    for doc in document_list:
        yield FetchAndParseRequest(
            fetcher_id='fs-fetcher',
            fetch_key=doc
        )

responses = stub.FetchAndParseBiDirectionalStreaming(request_generator())

for response in responses:
    print(f"Processed: {response.fetch_key} - {response.status}")
```

## Configuration

### Server Startup

```bash
java -jar tika-grpc.jar \
  --port 50051 \
  --config tika-config.xml \
  --numClients 10 \
  --maxMessageSize 104857600
```

### Configuration Options

- `--port`: gRPC server port (default: 50051)
- `--config`: Path to Tika configuration XML
- `--numClients`: Number of Tika Pipes clients
- `--maxMessageSize`: Max message size in bytes
- `--certChain`: Server certificate chain (mTLS)
- `--privateKey`: Server private key (mTLS)
- `--trustCert`: Trust certificate collection (mTLS)

### Tika Configuration

Standard Tika configuration XML with pipes configuration:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<properties>
  <pipes>
    <params>
      <param name="numClients" type="int">10</param>
      <param name="timeoutMillis" type="long">300000</param>
      <param name="maxForEmitBatchBytes" type="long">10000000</param>
    </params>
  </pipes>
</properties>
```

## Performance Considerations

### Connection Pooling
- Reuses gRPC channels
- Maintains Tika Pipes client pool
- Efficient resource utilization

### Streaming Benefits
- Reduced latency for batch operations
- Lower memory footprint
- Better throughput for large datasets

### Message Size Limits
Configure `maxMessageSize` based on document sizes:
- Default: 100MB
- Adjust for larger documents
- Consider compression for very large files

## Monitoring

### gRPC Metrics
Standard gRPC metrics available:
- Request counts
- Latency histograms
- Error rates
- Active connections

### Tika Pipes Metrics
Inherited from Tika Pipes:
- Parse success/failure rates
- Processing times
- Queue depths
- Worker utilization

## Testing

### Integration Tests
**Location**: `src/test/java/org/apache/tika/pipes/grpc/`

**Key Tests**:
- `TikaGrpcServerTest` - Basic functionality
- `PipesBiDirectionalStreamingIntegrationTest` - Streaming tests

### Test Setup
```java
TikaGrpcServer server = new TikaGrpcServer(port, configPath, 5);
server.start();
// Run tests
server.stop();
```

## Use Cases

### 1. Polyglot Microservices
Use Tika from non-Java services (Python, Go, Node.js)

### 2. High-Throughput Processing
Bi-directional streaming for maximum throughput

### 3. Dynamic Configuration
Update fetcher configurations without restart

### 4. Secure Processing
mTLS for authenticated, encrypted communication

### 5. Cloud Native Deployments
Deploy as containerized service in Kubernetes

## Comparison with tika-server

| Feature | tika-grpc | tika-server |
|---------|-----------|-------------|
| Protocol | gRPC/HTTP2 | REST/HTTP |
| Serialization | Protobuf | JSON/XML |
| Streaming | Native | Limited |
| Performance | Higher | Good |
| Client Support | Any language | HTTP clients |
| Binary Data | Efficient | Base64 encoded |
| Batch Processing | Bi-directional streams | Individual requests |

## Migration from tika-server

**Advantages of gRPC**:
- Better performance for batch operations
- Efficient binary serialization
- Native streaming support
- Stronger typing via protobuf

**When to Use Each**:
- **tika-grpc**: High-throughput batch processing, polyglot environments
- **tika-server**: Simple REST integrations, web browsers, debugging

## Related Documentation

- [Tika Pipes](../tika-pipes/overview.md)
- [Server Module](../tika-server/overview.md)
- [gRPC Protocol](https://grpc.io/)
- [Protocol Buffers](https://protobuf.dev/)

## Troubleshooting

### Connection Issues
- Check firewall rules for gRPC port
- Verify certificates for mTLS
- Check server logs for startup errors

### Performance Issues
- Increase `numClients` for more parallelism
- Adjust `maxMessageSize` for large documents
- Monitor Tika Pipes queue depth

### Configuration Errors
- Validate Tika configuration XML
- Check fetcher class names and configs
- Review JSON schema for fetcher configs

---

**Source Code**: `tika-grpc/`  
**Protocol Definition**: `tika-grpc/src/main/proto/tika.proto`  
**Since**: Tika 2.0  
**Latest Enhancements**: 4.0 (Dynamic component management)
