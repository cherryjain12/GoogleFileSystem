# snapshot.py

## Overview
`snapshot.py` implements the snapshot functionality for the Google File System. It allows creating point-in-time copies of directories, which can be used for backup, recovery, or creating consistent views of the file system.

## Key Functions

### send_json_data(ip, port, data)
Sends JSON data to a specified IP and port.

#### Parameters:
- `ip`: IP address of the target server
- `port`: Port of the target server
- `data`: JSON data to send

### find_nearest(self_ip, ips)
Finds the nearest IP address based on network proximity.

#### Parameters:
- `self_ip`: Reference IP address
- `ips`: List of IP addresses to check

#### Returns:
- The IP address from the list that is closest to `self_ip`

### takeSnap(metaData, dir_path, self_ip, self_port)
Creates a snapshot of a directory.

#### Parameters:
- `metaData`: Metadata object containing system state
- `dir_path`: Path of the directory to snapshot
- `self_ip`: IP address of the master server
- `self_port`: Port of the master server

## Snapshot Creation Process

1. Parse the directory path
2. Create a new SnapObj instance to store the snapshot
3. Retrieve the directory node from the namespace
4. Collect all file handles in the directory
5. For each file handle:
   - Copy the file metadata to the snapshot
   - Add the chunk handles to the snapshot's chunks database
6. Serialize the snapshot object
7. Select a chunk server to store the snapshot:
   - Get all available chunk servers
   - Find the nearest one to the master
8. Generate a timestamp for the snapshot
9. Create headers for the snapshot data
10. Send the snapshot data to the selected chunk server
11. Send the metadata mapping separately

## Data Structures

### SnapObj
A class imported from `dir_struct` that contains:
- `nameSpace`: Root of the snapshot directory tree
- `metadata`: List of file metadata in the snapshot
- `chunksDB`: List of chunks in the snapshot

## Error Handling
- Handles connection failures to chunk servers
- Processes directory paths with proper normalization
- Ensures all metadata is properly copied

## Dependencies
- `dir_struct`: For accessing the directory structure and SnapObj class
- `copy`: For deep copying objects
- `pickle`: For serializing objects
- `configparser`: For loading configuration
- `socket`: For network communication
- `time`: For generating timestamps

## Configuration
Uses parameters from `master.properties`:
- `DELIMITER`: Character used to separate message parts

## Usage
This module is called by the master server when a client requests a snapshot:

```python
# Example: Create a snapshot of a directory
snapshot.takeSnap(metaData, directory_path, master_ip, master_port)
```

## Restoration Process
While this file focuses on creating snapshots, restoration is handled by the master server when it receives a restore_snapshot request from a client:

1. Client sends restore_snapshot request with directory name
2. Master looks up the snapshot record
3. Master sends restore request to the chunk server hosting the snapshot
4. Chunk server sends the snapshot data back to the master
5. Master restores the directory structure and metadata
