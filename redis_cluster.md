```Docker
version: '3'

services:
  redis-cluster:
    environment:
      IP: "127.0.0.1z"
      # SENTINEL: ${REDIS_USE_SENTINEL}
      # STANDALONE: ${REDIS_USE_STANDALONE}
    image: grokzen/redis-cluster
    restart: on-failure
    ports:
      - "7000-7050:7000-7050"
      - "5000-5010:5000-5010"
    healthcheck:
      test: ["CMD", "redis-cli", "-c", "-h", "127.0.0.1", "-p", "7000"]
      interval: 30s
      timeout: 10s
      retries: 5
    networks:
      - redis-cluster-network
  # kafka:
    # image: "landoop/fast-data-dev"
    # environment:
    # - ADV_HOST=127.0.0.1
    # ports:
    #   - "2181:2181"
    #   - "3030:3030"
    #   - "8081:8081"
    #   - "8082:8082"
    #   - "9092:9092"
    # networks:
    #   - redis-cluster-network
  
networks:
  redis-cluster-network:
    driver: bridge 

```