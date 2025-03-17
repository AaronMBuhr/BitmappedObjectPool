# BitmappedObjectPool

A high-performance, thread-safe object pool implementation with bitmap-based allocation tracking and minimal memory overhead.

## Overview

BitmappedObjectPool is a C++ template library providing efficient memory management for scenarios requiring frequent object allocation and deallocation. It uses a bitmap-based tracking system to eliminate memory fragmentation and provide constant-time allocations.

The library consists of two main components:

1. **Bitmap**: A thread-safe, memory-efficient bit vector implementation
2. **BitmappedObjectPool**: A templated object pool using Bitmap to track allocations

## Features

### Bitmap
- Thread-safe operations with atomic bit counting
- Memory-efficient storage using fixed-size array
- Fast bit operations with constant-time complexity
- Support for finding first set/unset bit for allocation patterns
- Hex and binary string representation for debugging

### BitmappedObjectPool
- Type-safe template implementation for any C++ object type
- Chunk-based allocation with dynamic growth
- Controlled memory shrinking with configurable slack percentage
- Zero-fragmentation memory layout
- Thread-safe allocations for concurrent use
- O(1) allocation time for already-reserved memory
- Pointer validation to detect use-after-free bugs
- Support for cross-type conversion with proper type checking

## Usage

### Basic Example

```cpp
#include "BitmappedObjectPool.h"

// Create a pool of 1024 integers with 20% slack
BitmappedObjectPool<int> pool(1024, 20);

// Allocate a new integer
int* value = pool.getAndMarkNextUnused();
*value = 42;

// Use the value...

// Check if a pointer belongs to this pool
bool valid = pool.isValidObject(value);

// Return the integer to the pool
pool.markAsUnused(value);
```

### Advanced Usage

```cpp
// Create a specialized object pool
class MyObject {
public:
    MyObject() : value(0) {}
    void initialize(int val) { value = val; }
    int getValue() const { return value; }
private:
    int value;
};

// Create a pool with 512 objects per chunk and never shrink (slack = -1)
BitmappedObjectPool<MyObject> objectPool(512, -1);

// Allocate and initialize objects
MyObject* obj1 = objectPool.getAndMarkNextUnused();
obj1->initialize(100);

MyObject* obj2 = objectPool.getAndMarkNextUnused();
obj2->initialize(200);

// Count current allocations
int count = objectPool.countBuffers();  // Should be 2

// Return objects to the pool when done
objectPool.markAsUnused(obj1);
objectPool.markAsUnused(obj2);
```

## Memory Management

The pool allocates memory in chunks of the specified size. The `percent_slack` parameter controls when higher indexed chunks are freed, but only if all chunks above the current one are completely empty:

- `-1`: Never free chunks (always keep reserved memory)
- `0`: Free higher chunks as soon as they're completely empty
- `1-100`: Only free higher chunks when the current chunk has at least this percentage of unused slots

This approach prevents memory thrashing while allowing controlled release of memory when usage decreases.

## Thread Safety

All operations are thread-safe and can be called from multiple threads concurrently. The implementation uses mutexes and atomic operations to ensure data consistency.

## Performance Characteristics

- **Allocation**: O(1) for available bits in existing chunks
- **Deallocation**: O(1) in all cases
- **Memory overhead**: Approximately 1 bit per object plus chunk management structures
- **Fragmentation**: Zero fragmentation by design

## Requirements

- C++17 or later
- Standard library support for:
  - `std::mutex`
  - `std::atomic`
  - `std::shared_ptr`
  - `std::vector`
