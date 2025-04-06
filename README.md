# Google File System Implementation

This project is a Python implementation of the Google File System (GFS), a distributed file system designed for large-scale data processing workloads. The system follows a master-slave architecture with chunk-based storage and replication for fault tolerance.

## Architecture Overview

The implementation consists of three main components:

### 1. Master Server
- Central coordinator that manages metadata and system operations
- Maintains the file namespace (directory structure)
- Tracks chunk locations and their replicas
- Handles client requests for file operations
- Performs periodic health checks on chunk servers
- Manages replication and load balancing

### 2. Chunk Servers (Slaves)
- Store actual file data in chunks
- Report their status to the master periodically
- Handle read/write operations from clients
- Perform integrity checks on chunks
- Replicate chunks to other servers when needed

### 3. Client
- Interfaces with the master for metadata operations
- Communicates directly with chunk servers for data transfer
- Supports operations like read, write, delete, snapshot, etc.

## Key Features

### Chunk Management
- Files are divided into fixed-size chunks (configurable)
- Each chunk has a unique identifier (SHA-1 hash of content)
- Chunks are replicated across multiple chunk servers for redundancy
- The system maintains primary and secondary replicas

### Fault Tolerance
- Periodic health checks detect server failures
- Automatic re-replication of chunks when servers fail
- Integrity checking using checksums to detect corrupted chunks
- Recovery mechanisms for corrupted chunks

### Data Integrity
- Chunks are verified using SHA-1 checksums
- Corrupted chunks are detected and re-replicated from valid copies
- Block-level checksums for efficient integrity verification

### Snapshot and Recovery
- Support for creating snapshots of directories
- Ability to restore from snapshots
- Metadata versioning for consistent snapshots

### Load Balancing
- Distribution of chunks based on server capacity
- Rebalancing when new servers are added
- Prioritizing servers with more free space

## Project Structure

```
GoogleFileSystem-main/
├── app/
│   ├── master/                 # Master server implementation
│   │   ├── master.py           # Main master server code
│   │   ├── dir_struct.py       # Directory structure implementation
│   │   ├── ListenClientSlave.py # Communication handler
│   │   ├── reReplicateChunk.py # Chunk replication manager
│   │   ├── snapshot.py         # Snapshot functionality
│   │   ├── delete_file.py      # File deletion operations
│   │   ├── master.properties   # Master configuration
│   │   └── chunk_servers.json  # Chunk server registry
│   │
│   ├── client/                 # Client implementation
│   │   ├── client.py           # Client interface
│   │   └── client.properties   # Client configuration
│   │
│   └── slave_copies/           # Chunk server implementations
│       ├── slave1/             # First chunk server
│       │   ├── slave.py        # Chunk server code
│       │   └── slave.properties # Server configuration
│       ├── slave2/             # Second chunk server
│       ├── slave3/             # Third chunk server
│       └── slave4/             # Fourth chunk server
│
└── test/                       # Test utilities and examples
    ├── gen_random.py           # Random data generator
    ├── server.py               # Test server
    ├── client.py               # Test client
    ├── test.py                 # Test script
    └── divide and merge.py     # Chunk division utilities
```

## Data Structures

### Metadata Management
- **fileNamespace**: Tree structure representing the directory hierarchy
- **metadata**: Maps files to their chunks
- **chunksDB**: List of all chunks in the system
- **slaves_list**: Information about all chunk servers

### Chunk Mapping
- Maps chunk handles to their server locations
- Tracks the status of each replica (primary/secondary, valid/invalid)
- Maintains server load information

## Communication Protocol

The system uses TCP sockets for communication with a custom protocol:
- JSON-based messages for metadata operations
- Binary transfers for chunk data
- Delimiter-based headers for message parsing

## Workflow Examples

### File Creation
1. Client sends create request to master with file data
2. Master creates file entry in namespace
3. Master divides file into chunks and generates chunk handles
4. Master allocates chunk servers for each chunk
5. Master sends chunk data to selected servers
6. Chunk servers store the data and report back

### File Reading
1. Client requests file metadata from master
2. Master returns chunk handles and server locations
3. Client contacts chunk servers directly to read data
4. Chunk servers verify integrity before sending data
5. Client assembles chunks into complete file

### Snapshot Creation
1. Client requests snapshot of a directory
2. Master creates a copy of the directory's metadata
3. Master selects a chunk server to store the snapshot
4. Snapshot metadata is sent to the selected server
5. Snapshot record is maintained for future restoration

## Setup and Usage

### Prerequisites
- Python 3.x
- Network connectivity between master, chunk servers, and clients

### Configuration
1. Update the master.properties file with appropriate settings
2. Configure each slave.properties file with the correct master address
3. Set up client.properties with connection parameters

### Running the System
1. Start the master server:
   ```
   python master.py <ip:port>
   ```

2. Start chunk servers:
   ```
   python slave.py <ip:port> <master_ip:master_port>
   ```

3. Run the client:
   ```
   python client.py <ip:port> <master_ip:master_port>
   ```

### Client Commands
- `read`: Read a file with specified byte range
- `create`: Create a new file
- `delete`: Delete a file
- `snapshot`: Create a snapshot of a directory
- `restore_snapshot`: Restore a directory from a snapshot

## System Monitoring and Maintenance

- Periodic background tasks for metadata persistence
- Regular health checks on chunk servers
- Automatic rebalancing of chunks
- Garbage collection of orphaned chunks

## Implementation Details

The implementation follows the design principles outlined in the original Google File System paper, with adaptations for Python and simplified for educational purposes. Key aspects include:

- Separation of metadata and data paths
- Centralized master for metadata management
- Distributed chunk servers for scalable storage
- Replication for fault tolerance and availability
- Checksumming for data integrity
- Snapshot mechanism for point-in-time recovery
