# Kafka Multi-Node Environment

This project sets up a 3-node Apache Kafka cluster using KRaft (Kafka Raft) mode with Confluent Control Center for monitoring and management.

## Architecture Overview

The environment consists of:
- **3 Kafka Brokers** (`kafka_01`, `kafka_02`, `kafka_03`) running in combined broker/controller mode
- **Confluent Control Center** for cluster monitoring and management
- **KRaft Mode** (no ZooKeeper required) for metadata management

## Services

### Kafka Nodes (kafka_01, kafka_02, kafka_03)

Each Kafka node runs as both a broker and a controller in KRaft mode:

- **kafka_01**: 
  - Broker port: `9092`
  - Controller port: `29092`
  - Node ID: `1`

- **kafka_02**: 
  - Broker port: `9093`
  - Controller port: `29093`
  - Node ID: `2`

- **kafka_03**: 
  - Broker port: `9094`
  - Controller port: `29094`
  - Node ID: `3`

#### Key Configuration

- **Process Roles**: Each node serves as both `broker` and `controller`
- **Quorum Voters**: All three nodes participate in the controller quorum for consensus
- **Listeners**:
  - `CONTROLLER`: Internal port for controller communication (KRaft consensus)
  - `LISTENER_BROKER`: Port for client/broker communication
- **Cluster ID**: `nypjLRGA3GQBYyVb` (must be consistent across all nodes)

### Confluent Control Center

Web-based UI for managing and monitoring the Kafka cluster:

- **Port**: `9021` (HTTP interface)
- **Features**:
  - Cluster health monitoring
  - Topic management
  - Consumer group monitoring
  - Message browsing
  - Performance metrics

Access at: `http://localhost:9021`

## Network Architecture

All services communicate via the `kafka_network` bridge network:

- **Internal Communication**: Containers use hostnames (`kafka_01`, `kafka_02`, `kafka_03`)
- **External Access**: Host machine can connect to brokers via `localhost:9092`, `localhost:9093`, `localhost:9094`

## Data Persistence

Persistent volumes ensure data survives container restarts:

- `kafka_data_01`, `kafka_data_02`, `kafka_data_03`: Kafka logs and metadata
- `control_center_data`: Control Center configuration and state

## Getting Started

### Prerequisites

- Docker Engine
- Docker Compose v3.8+

### Start the Cluster

```bash
docker-compose up -d
```

This will start all services in detached mode.

### Verify Cluster Status

Check that all containers are running:

```bash
docker-compose ps
```

View logs for a specific service:

```bash
docker-compose logs -f kafka_01
```

### Access Control Center

Once all services are healthy (may take 1-2 minutes):

```bash
open http://localhost:9021
```

Or navigate to `http://localhost:9021` in your browser.

### Stop the Cluster

```bash
docker-compose down
```

To remove volumes as well (deletes all data):

```bash
docker-compose down -v
```

## Testing the Cluster

### Create a Topic

```bash
docker exec -it kafka_01 kafka-topics \
  --bootstrap-server kafka_01:9092 \
  --create \
  --topic test-topic \
  --partitions 3 \
  --replication-factor 3
```

### List Topics

```bash
docker exec -it kafka_01 kafka-topics \
  --bootstrap-server kafka_01:9092 \
  --list
```

### Produce Messages

```bash
docker exec -it kafka_01 kafka-console-producer \
  --bootstrap-server kafka_01:9092 \
  --topic test-topic
```

Type messages and press Enter. Use Ctrl+C to exit.

### Consume Messages

```bash
docker exec -it kafka_01 kafka-console-consumer \
  --bootstrap-server kafka_01:9092 \
  --topic test-topic \
  --from-beginning
```

## Connecting from Host Machine

To connect applications running on your host machine:

```properties
bootstrap.servers=localhost:9092,localhost:9093,localhost:9094
```

## High Availability

This setup provides:

- **Fault Tolerance**: Cluster continues operating if one node fails
- **Data Replication**: Default replication factor of 3 ensures no data loss
- **Controller Quorum**: Requires 2 out of 3 controllers for consensus (majority)

## Troubleshooting

### Containers Won't Start

Check logs for specific errors:
```bash
docker-compose logs kafka_01
```

Ensure ports 9092, 9093, 9094, 9021, 29092, 29093, 29094 are not in use.

### Control Center Unreachable

Wait 1-2 minutes after startup for initialization. Check status:
```bash
docker-compose logs control_center
```

### Reset Everything

To completely reset the cluster:
```bash
docker-compose down -v
docker-compose up -d
```

## Configuration Notes

- **KRaft Mode**: This setup uses KRaft (KIP-500) instead of ZooKeeper for metadata management
- **Cluster ID**: The `CLUSTER_ID` must be identical across all nodes and formatted correctly
- **Metrics Reporter**: Confluent Metrics Reporter sends metrics to Control Center for monitoring
- **Replication**: Control Center topics use replication factor of 3 for high availability

## Additional Resources

- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent Platform Documentation](https://docs.confluent.io/)
- [KRaft Mode Overview](https://kafka.apache.org/documentation/#kraft)
