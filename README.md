# This is my repository store docker use in my work

1. [Kafka cluster docker-compose](kafka_cluster.md)
2. [Redis cluster docker-compose](redis_cluster.md)
3. [Build distroless image for Golang](build_distroless_image_for_golang.md)

```mermaid
graph TD
    LIB --> |HTTP or gRPC| COLLECTOR
    LIB["Jaeger Client (deprecated)"] --> |UDP| AGENT[Jaeger Agent]
    %% AGENT --> |HTTP/sampling| LIB
    AGENT --> |gRPC| COLLECTOR[Jaeger Collector]
    %% COLLECTOR --> |gRPC/sampling| AGENT
    SDK["OpenTelemetry SDK (recommended)"] --> |UDP| AGENT
    SDK --> |HTTP or gRPC| COLLECTOR
    COLLECTOR --> STORE[Storage]
    COLLECTOR --> |gRPC| PLUGIN[Storage Plugin]
    PLUGIN --> STORE
    QUERY[Jaeger Query Service] --> STORE
    QUERY --> |gRPC| PLUGIN
    UI[Jaeger UI] --> |HTTP| QUERY
    subgraph Application Host
        subgraph User Application
            LIB
            SDK
        end
        AGENT
    end
```
