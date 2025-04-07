# Google File System Implementation - Technical Documentation

This directory contains comprehensive technical documentation for the Google File System implementation. Each file provides in-depth information about specific components, algorithms, data structures, and communication protocols used in the project.

## Architecture Diagram

See the [Architecture Diagram](architecture_diagram.md) for a visual representation of the system components and their interactions.

## Core Components

### Master Server
- [master.py](master_py.md): The central coordinator that manages metadata and system operations
- [dir_struct.py](dir_struct_py.md): Implements the directory structure and metadata management
- [ListenClientSlave.py](ListenClientSlave_py.md): Handles communication with clients and chunk servers
- [reReplicateChunk.py](reReplicateChunk_py.md): Manages chunk replication and load balancing
- [snapshot.py](snapshot_py.md): Implements snapshot functionality for directories
- [delete_file.py](delete_file_py.md): Handles file deletion operations

### Client
- [client.py](client_py.md): Implements the client interface for interacting with the system

### Chunk Server
- [slave.py](slave_py.md): Implements the chunk server functionality

### Testing
- [Test Files](test_files.md): Documentation for test utilities and examples

## System Architecture

The Google File System follows a master-slave architecture with three primary components:

### Master Server
The master server is the central coordinator responsible for:
- Maintaining the file namespace (directory structure)
- Tracking chunk locations and their replicas
- Managing metadata operations
- Coordinating chunk server activities
- Handling client requests for file operations
- Performing health checks and rebalancing

### Chunk Servers
Chunk servers are responsible for:
- Storing actual file data in fixed-size chunks
- Performing integrity checks on stored chunks
- Handling read/write operations from clients
- Replicating chunks to other servers when instructed
- Reporting status to the master periodically

### Client
The client component:
- Interfaces with the master for metadata operations
- Communicates directly with chunk servers for data transfer
- Implements intelligent server selection for optimal performance
- Handles file assembly and disassembly

## Core Data Structures

### Master Server Data Structures

#### File Namespace (`fileNamespace`)
- Implemented as a recursive tree structure in `dir_struct.py`
- Each node represents either a directory or a file
- Directory nodes contain:
  - `name`: Directory name
  - `children_ptr`: List of pointers to child nodes
  - `children_name`: List of names of child nodes
  - `isFile`: Boolean flag (always false for directories)
- File nodes contain:
  - `name`: File name
  - `isFile`: Boolean flag (true for files)
  - `fileHash`: SHA-1 hash identifier for the file

#### Metadata Mapping (`metadata`)
- List of file objects that map files to their chunks
- Each file object contains:
  - `fileHashName`: SHA-1 hash identifier for the file
  - `chunkDetails`: List of chunks that make up the file
    - Each chunk entry contains `chunk_handle` (SHA-1 hash) and `chunk_index` (position in file)

#### Chunks Database (`chunksDB`)
- List of all chunk handles (SHA-1 hashes) in the system
- Used for global chunk tracking and orphan detection

#### Chunk Mapping (`chunks_mapping`)
- Maps chunk handles to their server locations
- Each entry contains:
  - `chunk_handle`: SHA-1 hash identifier for the chunk
  - `servers`: List of servers hosting the chunk
    - Each server entry contains `ip`, `port`, `type` (primary/secondary), and `isValidReplica` flag

#### Slave State (`slaves_list`)
- Information about all chunk servers in the system
- Each entry contains:
  - `ip`: IP address of the chunk server
  - `port`: Port number of the chunk server
  - `disk_free_space`: Available storage capacity
  - `chunks`: List of chunk handles stored on this server

### Chunk Server Data Structures

#### Chunk State (`chunks_state`)
- Tracks the state of chunks stored on the server
- Each entry contains:
  - `handle`: Chunk handle (SHA-1 hash)
  - `isValid`: Boolean flag indicating chunk integrity
  - `type`: Role of this replica (primary/secondary)
  - `valid_data_len`: Actual data length in the chunk

#### Checksum Objects (`CHECKSUM_OBJ`)
- Stores checksums for integrity verification
- Each entry contains:
  - `chunk_handle`: Chunk handle (SHA-1 hash)
  - `check_sums`: List of SHA-1 hashes for each block in the chunk

## Communication Protocols

### Message Types

#### Master-Client Communication
- **File Creation**: Client sends file data to master, which coordinates chunk creation
- **File Reading**: Client requests chunk locations from master, then reads directly from chunk servers
- **File Deletion**: Client requests file deletion, master updates namespace and notifies chunk servers
- **Snapshot Creation**: Client requests directory snapshot, master coordinates metadata copying
- **Snapshot Restoration**: Client requests snapshot restoration, master coordinates metadata and chunk recovery

#### Master-Chunk Server Communication
- **Health Checks**: Master periodically polls chunk servers for status
- **Chunk Allocation**: Master instructs chunk servers to store new chunks
- **Replication Instructions**: Master coordinates chunk replication between servers
- **Load Balancing**: Master instructs chunk servers to redistribute chunks

#### Client-Chunk Server Communication
- **Chunk Reading**: Client reads chunk data directly from chunk servers
- **Chunk Writing**: Client writes chunk data directly to primary chunk server

### Protocol Format

#### JSON Messages
- Used for metadata operations and control messages
- Standard format:
  ```json
  {
    "agent": "[client|master|chunk_server]",
    "action": "[action_type]",
    "ip": "[sender_ip]",
    "port": "[sender_port]",
    "data": {
      // Action-specific data
    }
  }
  ```

#### Binary Transfers
- Used for chunk data transfer
- Format: `[header][chunk_data]`
- Header format: `[DELIMITER][action][DELIMITER][chunk_handle][DELIMITER][type][DELIMITER][data_length][DELIMITER]`
- Header is padded to a fixed size (typically 200 bytes)

## Key Algorithms

### Chunk Allocation Algorithm
1. Sort available chunk servers by free space (descending)
2. Select the top three servers for primary and secondary replicas
3. Allocate chunk to these servers with appropriate replica types
4. Update server load information

### Integrity Verification Algorithm
1. Divide chunk into fixed-size blocks (configurable)
2. Compute SHA-1 hash for each block
3. Store hashes in checksum object
4. During read operations, recompute hashes and compare with stored values
5. If mismatch detected, mark chunk as invalid and trigger re-replication

### Server Selection Algorithm (Client-side)
1. Compare client IP address with chunk server IP addresses
2. Calculate network proximity based on matching IP octets
3. Select server with highest proximity score
4. Fall back to other replicas if selected server is unavailable

### Load Balancing Algorithm
1. Periodically check chunk distribution across servers
2. Identify servers with significantly more chunks than average
3. Select chunks to move from overloaded servers
4. Instruct underloaded servers to fetch these chunks
5. Update chunk mapping after successful transfer

## End-to-End Lifecycle Examples

### Complete File Creation Lifecycle
1. **Client Initiates Request**:
   - Client reads file data from local file or user input
   - Client establishes TCP connection to master server
   - Client sends create request with file path and data

2. **Master Processes Request**:
   - Master parses the request and extracts file path and data
   - Master creates file entry in namespace tree
   - Master divides file data into fixed-size chunks

3. **Chunk Processing**:
   - For each chunk:
     - Master computes SHA-1 hash as chunk handle
     - Master selects chunk servers based on available space
     - Master adds chunk to metadata and chunksDB

4. **Chunk Distribution**:
   - For each chunk and selected servers:
     - Master establishes TCP connection to chunk server
     - Master sends chunk data with appropriate headers
     - Chunk server stores data and computes checksums
     - Chunk server adds chunk to its local state

5. **Completion**:
   - All chunks are distributed and stored
   - Master updates its metadata structures
   - Master sends confirmation to client
   - Client closes connection

### Complete File Reading Lifecycle
1. **Client Initiates Request**:
   - Client establishes TCP connection to master server
   - Client sends read request with file path and byte range

2. **Master Processes Request**:
   - Master looks up file in namespace tree
   - Master identifies chunks needed for requested byte range
   - Master retrieves chunk locations from chunk mapping

3. **Master Response**:
   - Master sends chunk handles and server locations to client
   - Client receives metadata and closes connection to master

4. **Data Retrieval**:
   - For each required chunk:
     - Client selects optimal chunk server based on network proximity
     - Client establishes TCP connection to selected chunk server
     - Client requests specific byte range within chunk

5. **Integrity Verification**:
   - Chunk server verifies chunk integrity using checksums
   - If integrity check passes, chunk server sends data to client
   - If integrity check fails, chunk server notifies master and client tries another replica

6. **Data Assembly**:
   - Client receives chunk data from multiple chunk servers
   - Client assembles chunks in correct order based on chunk indices
   - Client presents complete file data to user

## Background Processes

### Master Server Background Processes

#### Metadata Persistence (`BgSaveOperationLog`)
- Periodically saves master state to disk (every 10 seconds)
- Ensures metadata can be recovered after master restart
- Implementation in `master.py`

#### Chunk Server Health Monitoring (`BgPoolChunkServer`)
- Periodically polls chunk servers for status (every 5 seconds)
- Detects server failures and triggers re-replication
- Implementation in `master.py`

### Chunk Server Background Processes

#### Status Reporting
- Responds to master's health check requests
- Reports available disk space and hosted chunks
- Identifies and reports corrupted chunks

## Error Handling and Recovery

### Chunk Server Failure
1. Master detects failure through missed heartbeat responses
2. Master identifies chunks hosted on failed server
3. Master selects new servers for re-replication
4. Master instructs healthy servers hosting replicas to send copies to new servers
5. System returns to normal replication level

### Corrupted Chunk Detection
1. Chunk server detects corruption during integrity check
2. Chunk server marks chunk as invalid and notifies master
3. Master identifies valid replicas of the chunk
4. Master instructs a server with valid replica to send copy to corrupted server
5. Corrupted server replaces invalid chunk with valid copy

### Master Server Recovery
1. Master server starts up after failure
2. Master loads metadata state from persistent storage
3. Master initiates health check to all registered chunk servers
4. Chunk servers report their current state
5. Master reconciles its metadata with chunk server reports
6. Master initiates re-replication for under-replicated chunks

## Implementation Limitations and Considerations

1. **Single Master**: The system uses a single master server, creating a potential single point of failure
2. **In-Memory Metadata**: The master keeps all metadata in memory, limiting scalability
3. **Simplified Authentication**: The implementation lacks robust authentication mechanisms
4. **Network Partitioning**: Limited handling of network partition scenarios
5. **Concurrent Write Operations**: The implementation has simplified handling of concurrent writes

## Performance Considerations

1. **Chunk Size Tradeoffs**: Larger chunks reduce metadata overhead but increase minimum I/O size
2. **Replication Factor**: Higher replication improves availability but increases storage requirements
3. **Block Size for Checksums**: Smaller blocks provide finer-grained integrity checking but increase metadata size
4. **Network Topology Awareness**: Server selection based on IP matching is a simple approximation of network proximity

For detailed documentation on specific components, please refer to the individual module documentation files in this directory.
