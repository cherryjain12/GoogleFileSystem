# delete_file.py

## Overview
`delete_file.py` implements the file deletion functionality for the Google File System. It handles the removal of files from the namespace and the cleanup of associated chunks.

## Key Functions

### DeleteFile(file_path, metaData, master_ip, master_port)
Deletes a file from the file system.

#### Parameters:
- `file_path`: Path of the file to delete
- `metaData`: Metadata object containing system state
- `master_ip`: IP address of the master server
- `master_port`: Port of the master server

#### Returns:
- `True` if the file was successfully deleted
- `False` otherwise

## File Deletion Process

1. Parse the file path
2. Retrieve the file handle from the namespace
3. If the file exists:
   - Remove the file entry from the namespace
   - Find the file metadata in the metadata list
   - Collect all chunk handles associated with the file
   - For each chunk:
     - Find chunk servers that have the chunk
     - Send delete requests to those servers
     - Remove the chunk from the chunks database
     - Remove the chunk mapping
   - Remove the file metadata from the metadata list
   - Return success
4. If the file doesn't exist, return failure

## Chunk Cleanup

For each chunk associated with the deleted file:
1. Check if the chunk is used by other files
2. If not used elsewhere:
   - Send delete requests to all chunk servers that have the chunk
   - Remove the chunk from the chunks database
   - Remove the chunk mapping

## Error Handling
- Validates file existence before deletion
- Handles cases where chunks are shared between files
- Manages communication failures with chunk servers

## Dependencies
- `dir_struct`: For accessing the directory structure and chunk mapping
- `socket`: For network communication
- `json`: For parsing and creating JSON messages

## Usage
This module is called by the master server when a client requests a file deletion:

```python
# Example: Delete a file
success = delete_file.DeleteFile(file_path, metaData, master_ip, master_port)
if success:
    print("File successfully deleted")
else:
    print("Failed to delete file")
```

## Security Considerations
- The module does not implement access control or permissions
- Any client can delete any file in the system
- In a production environment, additional security measures would be needed
