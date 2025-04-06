# reReplicateChunk.py

## Overview
`reReplicateChunk.py` manages chunk replication and load balancing in the Google File System. It ensures that chunks are properly distributed across chunk servers and handles replication when servers fail or new servers are added.

## Key Functions

### isValidChunk(chunk_handle, server_ip, server_port)
Checks if a chunk on a specific server is valid.

#### Parameters:
- `chunk_handle`: The unique identifier of the chunk
- `server_ip`: IP address of the chunk server
- `server_port`: Port of the chunk server

#### Returns:
- `True` if the chunk is valid on the specified server
- `False` otherwise

### find_nearest(ips, self_ip)
Finds the nearest IP address based on network proximity.

#### Parameters:
- `ips`: List of IP addresses to check
- `self_ip`: Reference IP address

#### Returns:
- The IP address from the list that is closest to `self_ip`

### distribute_load(self_ip, self_port, target_ip, target_port, metaData, container, disk_free_space=0, task="new_added")
Distributes chunks across chunk servers to balance the load.

#### Parameters:
- `self_ip`: IP address of the master server
- `self_port`: Port of the master server
- `target_ip`: IP address of the target chunk server
- `target_port`: Port of the target chunk server
- `metaData`: Metadata object containing system state
- `container`: Semaphore for thread synchronization
- `disk_free_space`: Available disk space on the target server (optional)
- `task`: Type of load distribution task (default: "new_added")

#### Tasks:
- `new_added`: Distribute chunks to a newly added chunk server
- `old_removed`: Redistribute chunks from a failed server

## Load Balancing Algorithm

### For New Servers
1. Calculate the maximum capacity of the new server based on disk space
2. Calculate the average number of chunks per server in the system
3. Determine how many chunks to allocate to the new server (minimum of capacity and average)
4. Sort existing servers by number of chunks (descending)
5. Transfer chunks from the most loaded servers to the new server:
   - Select valid chunks from the most loaded server
   - Update chunk mapping to point to the new server
   - Update disk space information for both servers
   - Remove the chunk from the source server's list

### For Failed Servers
1. Identify all chunks that were on the failed server
2. For each chunk:
   - Find other valid replicas of the chunk
   - Select a target server based on load and proximity
   - Update chunk mapping to reflect the new replica location
   - Send replication request to the selected server

## Error Handling
- Validates chunk integrity before replication
- Ensures chunks are only replicated to servers with sufficient disk space
- Maintains the required number of replicas for each chunk

## Dependencies
- `configparser`: For loading configuration
- `dir_struct`: For accessing the directory structure and chunk mapping
- `socket`: For network communication
- `json`: For parsing and creating JSON messages

## Configuration
Uses parameters from `master.properties`:
- `CHUNKSIZE`: Size of chunks in bytes

## Usage
This module is primarily used by the master server in two scenarios:
1. When a new chunk server joins the system
2. When an existing chunk server fails

```python
# Example: Distribute load to a new server
reReplicateChunk.distribute_load(
    self_ip, 
    self_port, 
    new_server_ip, 
    new_server_port, 
    metaData, 
    container, 
    disk_free_space, 
    "new_added"
)

# Example: Redistribute chunks from a failed server
reReplicateChunk.distribute_load(
    self_ip, 
    self_port, 
    failed_server_ip, 
    failed_server_port, 
    metaData, 
    container, 
    task="old_removed"
)
```
