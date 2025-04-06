# dir_struct.py

## Overview
`dir_struct.py` implements the directory structure and metadata management for the Google File System. It provides the core data structures for tracking files, chunks, and their locations across the distributed system.

## Key Components

### Global Variables
- `globalChunkMapping`: A global instance of the ChunkLoc class that maps chunks to chunk servers
- `config`: Configuration parameters loaded from master.properties
- `CHUNKSIZE`: Size of chunks in bytes, loaded from configuration

### Classes

#### ChunkLoc
Manages the mapping between chunks and chunk servers.

##### Attributes:
- `chunks_mapping`: List of dictionaries mapping chunk handles to server locations
- `slaves_state`: List of dictionaries containing information about chunk servers

#### DumpObj
Container for persistent metadata.

##### Attributes:
- `fileNamespace`: Root of the directory tree
- `metadata`: List of file metadata (mapping files to chunks)
- `chunksDB`: List of all chunks in the system
- `slaves_list`: Information about all chunk servers
- `snapshotRecord`: Record of all snapshots

#### SnapObj
Container for snapshot metadata.

##### Attributes:
- `nameSpace`: Root of the snapshot directory tree
- `metadata`: List of file metadata in the snapshot
- `chunksDB`: List of chunks in the snapshot

#### Tree
Implements a tree structure for the file namespace.

##### Methods:
- `__init__(self, x="")`: Initialize a tree node with a name
- `allocateServers(self, container, metaObj)`: Allocate chunk servers for a new chunk
- `fillMetaData(self, file_name, file_hash, metaObj, container)`: Process a file into chunks and update metadata
- `traverseInsert(self, dir_path, tree_root, isFile, metaObj, container)`: Recursively traverse the tree to insert a new node
- `insert(self, name, isFile, tree_root, metaObj, container)`: Insert a new file or directory into the tree
- `showDirectoryStructure(self, tree_root)`: Display the directory structure
- `traverseTree(self, tree_root, fileName_arr)`: Traverse the tree to find a file
- `retrieveNode(self, tree_root, dir_arr)`: Retrieve a specific node in the tree
- `retrieveHandle(self, file_name)`: Get the handle for a file
- `retrieveAllFileHandles(self, tree_root, file_handles)`: Get all file handles in a directory
- `traverseRemove(self, tree_root, fileName_arr)`: Remove a file or directory from the tree
- `snapRemove(self, tree_root, fileName_arr)`: Remove a snapshot from the tree
- `mergeTrees(self, dir_path, tree_root, new_tree_root)`: Merge two directory trees
- `removeEntry(self, file_name)`: Remove a file entry from the tree

## Key Workflows

### Chunk Allocation
1. Sort chunk servers by available disk space
2. Select the top three servers for primary and secondary replicas
3. Update server disk space information
4. Return the selected servers

### File Processing
1. Read the file in chunks of size CHUNKSIZE
2. Generate a SHA-1 hash for each chunk
3. Allocate servers for each chunk
4. Send chunk data to the allocated servers
5. Update metadata with chunk information

### Directory Operations
- Insert: Add a new file or directory to the namespace
- Remove: Delete a file or directory from the namespace
- Traverse: Navigate the directory structure to find files
- Snapshot: Create a point-in-time copy of a directory

## Error Handling
- Handles connection failures to chunk servers
- Validates chunk server availability
- Manages disk space constraints

## Dependencies
- Standard libraries: `hashlib`, `json`, `configparser`, `socket`, `threading`, `copy`, `os`

## Usage
This module is primarily used by the master server to manage the file system's metadata and directory structure.
