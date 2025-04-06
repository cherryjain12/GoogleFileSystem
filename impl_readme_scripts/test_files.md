# Test Files

## Overview
The test directory contains utilities and examples for testing the Google File System implementation. These files help verify the functionality of the system and demonstrate its usage.

## Files

### gen_random.py
A utility for generating random data files for testing.

#### Key Functions:
- Generates files of specified sizes with random content
- Useful for testing file creation, reading, and chunk distribution

### server.py
A simple test server implementation.

#### Key Features:
- Simulates basic server functionality
- Useful for testing network communication

### client.py
A test client implementation.

#### Key Features:
- Simulates client requests
- Demonstrates how to interact with the GFS

### test.py
The main test script for the Google File System.

#### Key Features:
- Runs various test scenarios
- Verifies system functionality
- Reports test results

### divide and merge.py
Utilities for dividing files into chunks and merging them back.

#### Key Functions:
- `divide_file(file_path, chunk_size)`: Divides a file into chunks of specified size
- `merge_chunks(chunk_files, output_file)`: Merges chunks back into a complete file

## Usage

### Running Tests
```
python test.py
```

### Generating Test Data
```
python gen_random.py <size_in_bytes> <output_file>
```

### Testing File Division and Merging
```
python "divide and merge.py" <operation> <file_path> [chunk_size]
```
Where:
- `<operation>` is either "divide" or "merge"
- `<file_path>` is the path to the file to process
- `[chunk_size]` is the optional chunk size for division

## Test Scenarios

1. **Basic Functionality**
   - Create, read, and delete files
   - Verify data integrity

2. **Fault Tolerance**
   - Simulate chunk server failures
   - Verify automatic re-replication

3. **Snapshot and Recovery**
   - Create snapshots of directories
   - Restore from snapshots
   - Verify data consistency

4. **Load Balancing**
   - Add new chunk servers
   - Verify chunk redistribution

5. **Performance Testing**
   - Measure throughput for various operations
   - Test with different file sizes and chunk sizes

## Dependencies
- The main GFS implementation
- Standard Python libraries
