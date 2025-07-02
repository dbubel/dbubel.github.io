# Memory Mapped Files & Binary Protocols

```
                         ┌─────────────────────────────────────────────────┐
                         │  📄 data.bin                                    │
                         │                                                 │
                         │  ┌───────────────────────────────────────────┐  │
                         │  │ H e l l o   W o r l d !                   │  │
                         │  │                                           │  │
                         │  │ 0 1 1 0 1 0 1 1   1 0 0 1 1 1 0 0 1 0 1   │  │
                         │  │ 1 0 1 0 1 1 0 0   0 1 1 1 0 0 1 0 1 1 0   │  │
                         │  │ 0 1 0 1 0 1 1 1   1 0 1 0 1 1 0 1 0 0 1   │  │
                         │  │                                           │  │
                         │  │ [ more data blocks and binary content ]   │  │
                         │  │ [ structured records and metadata ]       │  │
                         │  │ [ event streams and time series data ]    │  │
                         │  └───────────────────────────────────────────┘  │
                         │                                                 │
                         │  📊 Size: 1,024 KB    📅 Modified: Today        │
                         │  🔒 Permissions: rw    🚀 Memory Mapped: Yes    │
                         └─────────────────────────────────────────────────┘
                                          Just bytes, but fast!
```

---































## What is a Memory Mapped File?

Memory mapping creates a bridge between your file and memory:

- **Direct mapping**: File contents appear as regular memory in your process
- **Zero-copy access**: No intermediate buffers or copying
- **OS managed**: The kernel handles loading/storing pages transparently
- **Simple interface**: Treat file data like a byte array

```
Traditional I/O:  File → Buffer → Process Memory
Memory Mapped:    File ←→ Process Memory (direct)
```

---






















## How It Works Under the Hood

```
Virtual Memory                 Physical Memory                File System
┌──────────────┐              ┌──────────────┐              ┌──────────────┐
│ 0x1000-0x2000│─────────────▶│ Page Frame A │◀─────────────│ data.bin     │
│   (4KB)      │              │    (4KB)     │              │  Block 0-4KB │
│              │              │              │              │              │
│ 0x2000-0x3000│─────────────▶│ Page Frame B │◀─────────────│ data.bin     │
│   (4KB)      │              │    (4KB)     │              │  Block 4-8KB │
│              │              │              │              │              │
│ 0x3000-0x4000│─────────────▶│ Page Frame C │◀─────────────│ data.bin     │
│   (4KB)      │              │    (4KB)     │              │  Block 8-12KB│
└──────────────┘              └──────────────┘              └──────────────┘
     ↑                             ↑                             ↑
 Program View                  Physical RAM                 Persistent Storage
```

**Key insight**: The OS maps virtual memory addresses directly to file blocks, eliminating the need for explicit read/write operations.

---


























## Performance: Why It's Fast
### System Call Elimination
```
Traditional I/O:    read() → kernel → user space (1000ns+)
Memory Mapped:      ptr[i] → CPU cache → register (1-10ns)
```

### The Magic Ingredients
- **Demand Paging**: Only load data when you access it
- **OS Prefetching**: Kernel reads ahead for sequential patterns  
- **CPU Cache Friendly**: Sequential access leverages L1/L2 cache
- **Shared Pages**: Multiple processes can share the same physical memory

---





















## Code Comparison

### Traditional I/O (System Call Heavy)
```go
// Every operation crosses the user→kernel boundary
file, _ := os.Open("data.txt")
buf := make([]byte, 1024)

n, _ := file.Read(buf)        // System call #1
file.Write([]byte("hello"))   // System call #2
```

### Memory Mapped (Direct Memory Access)
```go
mmap, err := syscall.Mmap(
    int(file.Fd()),    // file descriptor
    0,                 // offset  
    int(fileSize),     // size to map
    syscall.PROT_READ|syscall.PROT_WRITE,
    syscall.MAP_SHARED,
)

// Now these are just memory operations!
data := mmap[100]     // Direct read (no syscall)
mmap[200] = 42        // Direct write (no syscall)
```

---
























## Live Example: File Modification

```
Initial state:
File on disk:    [H][e][l][l][o][ ][W][o][r][l][d]
Memory mapped:   [H][e][l][l][o][ ][W][o][r][l][d]
Address:         0x1000            0x1005

Program execution:
mapped[6] = 'G'  // Change 'W' to 'G'
mapped[8] = '!'  // Change 'r' to '!'

Immediately after:
Memory mapped:   [H][e][l][l][o][ ][G][o][!][l][d]
File on disk:    [H][e][l][l][o][ ][W][o][r][l][d]  ← Not synced yet

After sync/close:
File on disk:    [H][e][l][l][o][ ][G][o][!][l][d]  ← Now matches

Result: "Hello World" → "Hello Go!ld"
```

---


























## Binary Protocols: The Perfect Partner

### Why Binary Beats Text
```
JSON:     {"timestamp": 1640995200000, "data": "hello"}  // 45 bytes
Binary:   [8 bytes timestamp][4 bytes size][5 bytes data] // 17 bytes
                                                         // 2.6x smaller!
```

**Benefits**: Smaller size, faster parsing, type safety, precise alignment

---

























## Event Stream Protocol Design

### File Structure
```go
// Stream Header (24 bytes) - written once
type StreamHeader struct {
    WriteOffset    int64 // Current write position  
    EventCount     int64 // Total events stored
    StartTimestamp int64 // Stream creation time
}

// Event Entry (16 bytes + data)
type EventEntry struct {
    Timestamp int64  // Unix nanoseconds
    Size      uint32 // Data length  
    Checksum  uint32 // CRC32 validation
    // Variable-length data follows
}
```

### Sequential Layout Strategy
```
┌─────────────┬─────────────┬─────────────┬─────────────┐
│ Stream      │ Event 1     │ Event 2     │ Event 3     │
│ Header      │ [16B header]│ [16B header]│ [16B header]│
│ (24 bytes)  │ [data]      │ [data]      │ [data]      │
└─────────────┴─────────────┴─────────────┴─────────────┘
```

**Why sequential?** Maximum space utilization + cache-friendly access patterns

---








RESULTS--------------------------










































## Real-World Performance Results

**Test**: 10,000 messages × 1KB each = 10MB total

| Method                   | Throughput (msgs/sec) | Throughput (MB/s) | vs Baseline   |
|--------------------------|----------------------:|------------------:|--------------:|
| **Zig Memory Mapped**    | 5,649,347             | 5,777             | **54.4x** ⚡  |
| **Memory Mapped**        | 3,984,986             | 4,081             | **38.4x**     |
| **Buffered Sequential**  | 1,041,608             | 1,067             | **10.0x**     |
| **Channel Synchronized** | 726,526               | 744               | **7.0x**      |
| **Standard Sequential**  | 103,865               | 106               | 1.0x          |

**Key insight**: Memory mapping delivers 38x performance improvement over standard I/O!

---


























## When to Use Memory Mapped Files

### ✅ Great For
- **Log processing**: Scan large files without loading into RAM
- **Database systems**: Efficient random access to data pages  
- **Inter-process communication**: Share data between processes
- **Data pipelines**: High-throughput sequential processing
- **Working with files larger than available RAM**
- **Durability**

### ❌ Avoid When  
- Small, infrequent I/O operations
- Mobile/embedded systems with limited virtual memory

---




















## Production Considerations

### Key Limitations
- **Crash safety**: No atomic operations, corruption possible during crashes
- **Error handling**: Memory access errors become SIGBUS/SIGSEGV signals
- **Memory pressure**: Large mappings can impact garbage collector performance
- **Platform differences**: Windows vs Unix behavior varies
- **NOT thread safe for writes***: Write operations must be synchronized to avoid data corruption

---




















## Key Takeaways

### The Performance Story
1. **System calls are expensive** - eliminate them for hot paths
2. **Sequential access wins** - leverage CPU cache and OS prefetching  
3. **Binary protocols** provide significant advantages over text formats
4. **Memory mapping scales** - handle TB files on GB RAM systems























