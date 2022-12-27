```Docker
version: "3"
services:
  zookeeper:
    image: docker.io/bitnami/zookeeper:3.8
    ports:
      - "2181:2181"
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
    volumes:
      - zookeeper_data:/bitnami/zookeeper
    restart: always

  kafka0:
    image: docker.io/bitnami/kafka:3.2
    ports:
      - "9080:9080"
    environment:
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_CFG_BROKER_ID=0
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9080
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka0:9092,EXTERNAL://localhost:9080
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_OPTS=-Xms256m -XX:MaxPermSize=128m

    volumes:
      - kafka_0_data:/bitnami/kafka
    depends_on:
      - zookeeper
    restart: always

  kafka1:
    image: docker.io/bitnami/kafka:3.2
    ports:
      - "9081:9081"
    environment:
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_CFG_BROKER_ID=1
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9081
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka1:9092,EXTERNAL://localhost:9081
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_OPTS=-Xms256m -XX:MaxPermSize=128m

    volumes:
      - kafka_1_data:/bitnami/kafka
    depends_on:
      - zookeeper
    restart: always


  kafka2:
    image: docker.io/bitnami/kafka:3.2
    ports:
      - "9082:9082"
    environment:
      - ALLOW_PLAINTEXT_LISTENER=yes
      - KAFKA_INTER_BROKER_LISTENER_NAME=CLIENT
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
      - KAFKA_CFG_BROKER_ID=2
      - KAFKA_CFG_LISTENER_SECURITY_PROTOCOL_MAP=CLIENT:PLAINTEXT,EXTERNAL:PLAINTEXT
      - KAFKA_CFG_LISTENERS=CLIENT://:9092,EXTERNAL://:9082
      - KAFKA_CFG_ADVERTISED_LISTENERS=CLIENT://kafka2:9092,EXTERNAL://localhost:9082
      - KAFKA_CFG_AUTO_CREATE_TOPICS_ENABLE=true
      - KAFKA_OPTS=-Xms256m -XX:MaxPermSize=128m

    volumes:
      - kafka_2_data:/bitnami/kafka
    depends_on:
      - zookeeper
    restart: always


#  kafka-init:
#    image: docker.io/bitnami/kafka:3.2
#    volumes:
#      - kafka_2_data:/bitnami/kafka
#      - ./msg.sh:/opt/bitnami/kafka/bin
#    depends_on:
#      - zookeeper
#      - kafka0
#      - kafka1
#      - kafka2
#    entrypoint: ["sh", "-c"]
#    command:
#      - "
#      kafka-topics.sh --create --if-not-exists --bootstrap-server kafka0:9092 --partitions 1 --replication-factor 1 --topic dev_user_recurring
#
#      "
  kafdrop:
    image: obsidiandynamics/kafdrop
    ports:
      - "9000:9000"
    environment:
      - KAFKA_BROKERCONNECT=kafka0:9092,kafka1:9092,kafka2:9092
    depends_on:
      - kafka0
      - kafka1
      - kafka2
    mem_limit: 400m
    mem_reservation: 384M

  redpanda:
    image: docker.redpanda.com/vectorized/console
    ports:
      - "8088:8080"
    environment:
      - KAFKA_BROKERS=kafka0:9092,kafka1:9092,kafka2:9092
    volumes:
      - ./data:/etc/console-mounts
    mem_limit: 400m
    mem_reservation: 384M

volumes:
  zookeeper_data:
    driver: local
  kafka_0_data:
    driver: local
  kafka_1_data:
    driver: local
  kafka_2_data:
    driver: local

```