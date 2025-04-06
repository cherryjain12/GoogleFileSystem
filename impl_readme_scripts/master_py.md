# master.py

## Overview
`master.py` is the core component of the Google File System implementation. It serves as the central coordinator for the entire distributed file system, managing metadata, chunk allocation, and system operations.

## Key Responsibilities
- Initialize and maintain the file system's metadata
- Handle connections from clients and chunk servers
- Coordinate chunk allocation and replication
- Perform periodic health checks on chunk servers
- Manage the file namespace (directory structure)
- Persist system state for recovery

## Classes

### BgSaveOperationLog
A background thread that periodically saves the master's state to disk.

#### Methods:
- `__init__(self, metaData, interval=10)`: Initializes the background thread with metadata and save interval
- `run(self)`: Continuously dumps the master's state to disk at specified intervals

### BgPoolChunkServer
A background thread that periodically checks the health of chunk servers.

#### Methods:
- `__init__(self, metaData, container, self_ip_port, interval=5)`: Initializes the thread with metadata and connection info
- `run(self)`: Continuously sends health check requests to all chunk servers

## Main Workflow
1. Load existing master state from disk or initialize a new state
2. Set up the directory structure if starting fresh
3. Start background threads for state persistence and chunk server health checks
4. Listen for incoming connections from clients and chunk servers
5. Create a new thread for each connection to handle requests

## Key Data Structures
- `metaData`: Contains the file namespace, metadata mapping, chunks database, and slave list
- `dir_struct.globalChunkMapping`: Global mapping of chunks to chunk servers

## Error Handling
- Gracefully handles missing configuration files
- Recovers from previous state when available
- Detects and handles chunk server failures

## Dependencies
- `dir_struct`: For directory structure and metadata management
- `reReplicateChunk`: For handling chunk replication
- `ListenClientSlave`: For handling client and chunk server connections
- Standard libraries: `threading`, `pickle`, `socket`, `json`, `configparser`

## Configuration
Uses `master.properties` for configuration parameters:
- `JSON_RCV_LIMIT`: Maximum size for JSON messages
- `CHUNKSIZE`: Size of chunks in bytes
- `DELIMITER`: Character used to separate message parts

## Usage
```
python master.py <ip:port>
```
Where `<ip:port>` is the IP address and port the master server should listen on.
