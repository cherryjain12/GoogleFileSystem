# Google File System Architecture Diagram

```
+-------------------------------------------------------------------------------------------+
|                                                                                           |
|                                     CLIENT                                                |
|                                                                                           |
+----------------+------------------------+----------------------------------------------+--+
                 |                        |                                              |
                 | Metadata               | Metadata                                     | Data
                 | Operations             | Results                                      | Transfer
                 |                        |                                              |
                 v                        |                                              v
+----------------+------------------------v----------------------------------------------+--+
|                                                                                           |
|                                     MASTER SERVER                                         |
|                                                                                           |
|  +-------------------+  +-------------------+  +-------------------+  +----------------+  |
|  |                   |  |                   |  |                   |  |                |  |
|  |  File Namespace   |  |  Chunk Mapping    |  |  Metadata Store   |  |  Chunk Server  |  |
|  |  (Tree Structure) |  |  (Chunk -> Server)|  |  (File -> Chunks) |  |  Manager       |  |
|  |                   |  |                   |  |                   |  |                |  |
|  +-------------------+  +-------------------+  +-------------------+  +----------------+  |
|                                                                                           |
|  +-------------------+  +-------------------+  +-------------------+                      |
|  |                   |  |                   |  |                   |                      |
|  |  Health Monitor   |  |  Load Balancer    |  |  Snapshot Manager |                      |
|  |  (BgPoolChunkSrv) |  |  (reReplicateChk) |  |  (snapshot.py)    |                      |
|  |                   |  |                   |  |                   |                      |
|  +-------------------+  +-------------------+  +-------------------+                      |
|                                                                                           |
+---------------------------+------------------------------+------------------------------+--+
                            |                              |                              |
                            | Heartbeat                    | Chunk                        | Chunk
                            | & Status                     | Operations                   | Operations
                            |                              |                              |
                            v                              v                              v
+---------------------------+------------------------------+------------------------------+--+
|                           |                              |                              |  |
|   CHUNK SERVER 1          |     CHUNK SERVER 2           |     CHUNK SERVER 3           |  |
|                           |                              |                              |  |
|  +-------------------+    |    +-------------------+     |    +-------------------+     |  |
|  | Chunk Storage     |    |    | Chunk Storage     |     |    | Chunk Storage     |     |  |
|  | (Chunks + Checksum|    |    | (Chunks + Checksum|     |    | (Chunks + Checksum|     |  |
|  |  Objects)         |    |    |  Objects)         |     |    |  Objects)         |     |  |
|  +-------------------+    |    +-------------------+     |    +-------------------+     |  |
|                           |                              |                              |  |
|  +-------------------+    |    +-------------------+     |    +-------------------+     |  |
|  | Integrity Checker |    |    | Integrity Checker |     |    | Integrity Checker |     |  |
|  | (SHA-1 Checksums) |    |    | (SHA-1 Checksums) |     |    | (SHA-1 Checksums) |     |  |
|  +-------------------+    |    +-------------------+     |    +-------------------+     |  |
|                           |                              |                              |  |
|  +-------------------+    |    +-------------------+     |    +-------------------+     |  |
|  | Chunk Replicator  |    |    | Chunk Replicator  |     |    | Chunk Replicator  |     |  |
|  | (Replica Manager) |    |    | (Replica Manager) |     |    | (Replica Manager) |     |  |
|  +-------------------+    |    +-------------------+     |    +-------------------+     |  |
|                           |                              |                              |  |
+---------------------------+------------------------------+------------------------------+--+

```

## Communication Flow

1. **Client-Master Communication**:
   - Client sends metadata requests to Master (file creation, lookup, deletion)
   - Master responds with metadata results (chunk locations, operation status)

2. **Client-Chunk Server Communication**:
   - Client reads/writes data directly from/to Chunk Servers
   - Chunk Servers verify data integrity before serving requests

3. **Master-Chunk Server Communication**:
   - Master sends chunk operation instructions to Chunk Servers
   - Chunk Servers send heartbeat and status reports to Master
   - Master coordinates chunk replication between Chunk Servers

## Data Flow

### File Creation Flow
```
Client -> Master: Create file request with data
Master: Creates file entry in namespace
Master: Divides file into chunks
Master: Allocates chunk servers for each chunk
Master -> Chunk Servers: Sends chunk data
Chunk Servers: Store chunks and compute checksums
Chunk Servers -> Master: Acknowledge storage
Master -> Client: Confirm file creation
```

### File Reading Flow
```
Client -> Master: Read file request
Master: Looks up file in namespace
Master: Identifies required chunks and locations
Master -> Client: Returns chunk locations
Client -> Chunk Servers: Requests chunk data
Chunk Servers: Verify chunk integrity
Chunk Servers -> Client: Send chunk data
Client: Assembles chunks into complete file
```

### Chunk Server Failure Recovery Flow
```
Master: Detects chunk server failure via missed heartbeat
Master: Identifies chunks that need re-replication
Master: Selects source and target servers for each chunk
Master -> Source Servers: Requests chunk replication
Source Servers -> Target Servers: Send chunk copies
Target Servers -> Master: Acknowledge replication
Master: Updates chunk mapping
```

## Key Components and Responsibilities

### Master Server
- **File Namespace**: Maintains directory structure as a tree
- **Chunk Mapping**: Tracks which servers store which chunks
- **Metadata Store**: Maps files to their constituent chunks
- **Chunk Server Manager**: Coordinates chunk server activities
- **Health Monitor**: Periodically checks chunk server status
- **Load Balancer**: Distributes chunks evenly across servers
- **Snapshot Manager**: Handles directory snapshot operations

### Chunk Server
- **Chunk Storage**: Stores actual file data in chunks
- **Integrity Checker**: Verifies chunk integrity using checksums
- **Chunk Replicator**: Handles chunk replication operations

### Client
- **Metadata Operations**: Communicates with master for metadata
- **Data Transfer**: Communicates directly with chunk servers for data
- **Server Selection**: Chooses optimal chunk server for data access
- **Data Assembly**: Combines chunks into complete files
