# ListenClientSlave.py

## Overview
`ListenClientSlave.py` implements the communication handler for the master server, processing requests from both clients and chunk servers. It serves as the interface between the master server and other components of the Google File System.

## Key Components

### Classes

#### ListenClientChunkServer
A thread class that handles incoming connections from clients and chunk servers.

##### Methods:
- `__init__(self, metaData, sock, self_ip, self_port, container)`: Initialize with metadata and connection info
- `send_json_data(self, ip, port, data)`: Send JSON data to a specified IP and port
- `find_nearest(self, ips)`: Find the nearest IP address based on network proximity
- `run(self)`: Main method that processes incoming requests

## Request Handling

### Client Requests

#### Read Operation
1. Receive file name and byte range from client
2. Look up file handle in the namespace
3. Find chunks that contain the requested data
4. Identify chunk servers that have those chunks
5. Send chunk information back to the client

#### Snapshot Operation
1. Receive directory path from client
2. Forward request to the snapshot module
3. Create a snapshot of the specified directory

#### Restore Snapshot Operation
1. Receive directory name from client
2. Look up snapshot information
3. Send restore request to the chunk server hosting the snapshot

#### Delete File Operation
1. Receive file path from client
2. Forward request to the delete_file module
3. Remove file from namespace and update metadata
4. Send response back to client

### Chunk Server Requests

#### Report Acknowledgment
1. Receive chunk server status report
2. Validate the chunk server
3. Process chunk information:
   - Update chunk mapping for valid chunks
   - Identify orphaned chunks
   - Update server disk space information
4. Send response with orphaned chunks list

#### Manipulated Chunk Found
1. Receive notification of corrupted chunk
2. Mark the chunk as invalid in the chunk mapping
3. Find valid replicas of the chunk
4. Select a server to seed the chunk to the affected server
5. Send seeding request to the selected server

#### Restoration Confirmation
1. Receive confirmation that a chunk has been restored
2. Update the chunk mapping to mark the replica as valid

#### New Chunk Server
1. Receive notification of a new chunk server
2. Wait for all existing chunk servers to report their status
3. Distribute load to the new server

## Binary Data Handling
The module also handles binary data transfers for file creation:
1. Parse headers from the binary data
2. Extract file content
3. Create a file in the namespace
4. Process the file into chunks

## Error Handling
- Handles connection failures gracefully
- Processes both JSON and binary data formats
- Validates chunk server identity
- Manages corrupted chunks detection and recovery

## Dependencies
- `json`: For parsing and creating JSON messages
- `socket`: For network communication
- `configparser`: For loading configuration
- `dir_struct`: For accessing the directory structure
- `delete_file`: For file deletion operations
- `reReplicateChunk`: For chunk replication
- `snapshot`: For snapshot operations

## Configuration
Uses parameters from `master.properties`:
- `JSON_RCV_LIMIT`: Maximum size for JSON messages
- `DELIMITER`: Character used to separate message parts
- `CHUNK_RECSIZE`: Maximum size for receiving chunks

## Usage
This class is instantiated by the master server for each incoming connection:
```python
listenthread = ListenClientChunkServer(metaData, conn, self_ip_port[0], int(self_ip_port[1]), container)
listenthread.daemon = True
listenthread.start()
```
