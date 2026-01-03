# Weather Station Data Service

This repository features a high-availability data ingestion system designed to handle worldwide weather metrics from the National Oceanic and Atmospheric Administration (NOAA). Using a distributed Cassandra cluster, the system manages real-time temperature updates via gRPC and ensures data integrity through strategic consistency level configurations.

## Key Technical Features

### 1. NoSQL Data Modeling
* **Partitioning Strategy**: Data is partitioned by Station ID to ensure even distribution across the cluster.
* **Time-Series Optimization**: Utilized `date` as a clustering key (ascending) for efficient range queries.
* **Schema Design**: Implemented User-Defined Types (UDT) for temperature records and `static` columns for station metadata to reduce storage redundancy.

### 2. Distributed Consistency Strategy ($R + W > RF$)
To handle the trade-offs between availability and consistency:
* **High-Availability Writes**: Configured `ConsistencyLevel.ONE` for data ingestion, ensuring sensors can always upload data even during partial cluster failure.
* **Strict Consistency Reads**: Enforced `ConsistencyLevel.TWO/THREE` for queries to prevent stale data retrieval, adhering to the $R + W > RF$ quorum rule.

### 3. Spark-to-Cassandra ETL
* Integrated **Apache Spark** to preprocess raw NOAA metadata (`ghcnd-stations.txt`).
* Performed distributed filtering and transformation of station records before batch-loading them into Cassandra.

### 4. Resilient gRPC API
* Built a gRPC server using **Protobuf** for structured communication.
* Implemented automated error handling for `Unavailable` and `NoHostAvailable` exceptions to ensure graceful degradation during node outages.

## Tech Stack
* **Database**: Apache Cassandra (3-Node Cluster)
* **Compute**: Apache Spark
* **API**: gRPC / Protocol Buffers
* **Containerization**: Docker & Docker Compose
* **Language**: Python (PySpark, Cassandra-Driver)

## Project Structure
* `server.py`: Core service logic and Cassandra session management.
* `station.proto`: Service definitions for gRPC communication.
* `docker-compose.yml`: Orchestration for the 3-node Cassandra cluster.

## Setup & Run
1. **Launch cluster**:
   ```bash 
   export PROJECT=p6 
   docker compose up -d

2. **Generate stubs**:
   ```bash 
   docker exec p6-db-1 sh -c "python3 -m grpc_tools.protoc -I=. --python_out=. --grpc_python_out=. station.proto"

3. **Start server**:
   ```bash 
   docker exec -it p6-db-1 python3 server.py


