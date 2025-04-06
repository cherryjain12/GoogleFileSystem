# client.py

## Overview
`client.py` implements the client interface for the Google File System. It allows users to interact with the distributed file system through various operations like reading, creating, deleting files, and managing snapshots.

## Key Components

### Classes

#### ListenMasterChunkServer
A thread class that handles incoming responses from the master and chunk servers.

##### Methods:
- `__init__(self, sock, ip, port)`: Initialize with socket and connection info
- `find_nearest(self, ips)`: Find the nearest IP address based on network proximity
- `run(self)`: Process incoming responses from master and chunk servers

#### TakeUserInput
A thread class that handles user input and sends requests to the master server.

##### Methods:
- `__init__(self, master_ip_port, self_ip_port)`: Initialize with master and client connection info
- `run(self)`: Continuously take user input and send appropriate requests

## Supported Operations

### Read Operation
1. User provides file name and byte range
2. Client calculates chunk indices based on the byte range
3. Client sends read request to master
4. Master responds with chunk handles and server locations
5. Client contacts chunk servers directly to read data
6. Chunk servers send the requested data
7. Client assembles the data and saves it locally

### Create Operation
1. User provides file name and data source (file or direct input)
2. Client reads the data and prepares it for transmission
3. Client sends create request with file data to master
4. Master creates the file, divides it into chunks, and distributes them
5. Master confirms successful creation

### Delete Operation
1. User provides file path
2. Client sends delete request to master
3. Master deletes the file and its chunks
4. Master confirms successful deletion

### Snapshot Operations
1. User provides directory path
2. Client sends snapshot request to master
3. Master creates a snapshot of the directory
4. For restoration, user provides directory name
5. Client sends restore request to master
6. Master restores the directory from the snapshot

## User Interface
The client provides a simple command-line interface:
- `read`: Read a file with specified byte range
- `create`: Create a new file
- `delete`: Delete a file
- `snapshot`: Create a snapshot of a directory
- `restore_snapshot`: Restore a directory from a snapshot

## Data Handling
- Uses a global `indices_arr` to track byte ranges for chunk requests
- Receives and processes both JSON and binary data
- Saves received file data to disk

## Error Handling
- Handles connection failures to master and chunk servers
- Processes both JSON and binary responses
- Validates user input

## Dependencies
- `socket`: For network communication
- `threading`: For concurrent processing
- `sys`: For command-line arguments
- `configparser`: For loading configuration
- `json`: For parsing and creating JSON messages
- `copy`: For deep copying objects

## Configuration
Uses parameters from `client.properties`:
- `CHUNK_RECSIZE`: Maximum size for receiving chunks
- `CHUNKSIZE`: Size of chunks in bytes
- `DELIMITER`: Character used to separate message parts

## Usage
```
python client.py <ip:port> <master_ip:master_port>
```
Where:
- `<ip:port>` is the IP address and port the client should listen on
- `<master_ip:master_port>` is the IP address and port of the master server
