# Google File System Implementation - Detailed Documentation

This directory contains detailed documentation for each component of the Google File System implementation. Each file provides in-depth information about a specific Python module in the project.

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

The Google File System implementation follows a master-slave architecture:

1. **Master Server**: Central coordinator that manages metadata
2. **Chunk Servers**: Store actual file data in chunks
3. **Client**: Interfaces with the master for metadata and chunk servers for data

## Key Workflows

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
