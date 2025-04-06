# slave.py

## Overview
`slave.py` implements the chunk server functionality for the Google File System. Chunk servers store the actual file data in chunks, handle read/write operations, perform integrity checks, and replicate chunks to other servers when needed.

## Key Components

### Global Variables
- `OK_REPORT`: Flag indicating if the server has reported to the master
- `CHECKSUM_OBJ`: List of checksums for chunks
- `container`: Semaphore for thread synchronization
- `chunks_state`: List of chunk states

### Classes

#### ListenClientMaster
A thread class that handles incoming requests from clients and the master server.

##### Methods:
- `__init__(self, sock, self_ip, self_port)`: Initialize with socket and connection info
- `send_json_data(self, ip, port, data)`: Send JSON data to a specified IP and port
- `generate_checkSum(self, file_name)`: Generate checksums for a file
- `check_integrity(self, start_byte, end_byte, chunk_handle)`: Verify chunk integrity
- `check_send_data(self, start_byte, end_byte, handle, clients_ip, clients_port)`: Check integrity and send data
- `replicate_chunks(self, handle, chunk_type, ip, port)`: Replicate chunks to other servers
- `run(self)`: Process incoming requests

## Key Operations

### Chunk Storage
1. Receive chunk data from the master
2. Extract chunk handle, type, and data length from headers
3. Save the chunk data to disk
4. Generate and store checksums for the chunk
5. Update the chunk server state

### Chunk Reading
1. Receive read request from a client
2. Verify the integrity of the requested chunk
3. If integrity check passes:
   - Read the requested byte range from the chunk
   - Send the data to the client
4. If integrity check fails:
   - Mark the chunk as invalid
   - Notify the master of the corrupted chunk
   - Wait for the chunk to be restored
   - Retry the read operation

### Chunk Replication
1. Receive replication request from the master
2. Verify the integrity of the chunk to be replicated
3. If integrity check passes:
   - Read the chunk data
   - Send it to the target server
4. If integrity check fails:
   - Mark the chunk as invalid
   - Notify the master of the corrupted chunk
   - Wait for the chunk to be restored
   - Retry the replication

### Health Reporting
1. Receive periodic health check request from the master
2. Collect information about stored chunks
3. Calculate available disk space
4. Send report to the master
5. Process response (e.g., delete orphaned chunks)

## Data Integrity
- Chunks are divided into blocks for efficient integrity checking
- SHA-1 checksums are generated for each block
- Integrity is verified before any read or replication operation
- Corrupted chunks are detected and reported to the master

## Error Handling
- Handles connection failures gracefully
- Detects and reports corrupted chunks
- Manages disk space constraints
- Processes both JSON and binary data formats

## Dependencies
- `threading`: For concurrent processing
- `socket`: For network communication
- `sys`: For command-line arguments
- `os`: For file system operations
- `time`: For sleep operations
- `json`: For parsing and creating JSON messages
- `configparser`: For loading configuration
- `hashlib`: For generating checksums
- `shutil`: For file operations

## Configuration
Uses parameters from `slave.properties`:
- `CHUNKSIZE`: Size of chunks in bytes
- `CHUNK_RECSIZE`: Maximum size for receiving chunks
- `DELIMITER`: Character used to separate message parts
- `BLOCK_SIZE`: Size of blocks for checksumming
- `MASTER_IP`: IP address of the master server
- `MASTER_PORT`: Port of the master server

## Usage
```
python slave.py <ip:port> <master_ip:master_port>
```
Where:
- `<ip:port>` is the IP address and port the chunk server should listen on
- `<master_ip:master_port>` is the IP address and port of the master server
