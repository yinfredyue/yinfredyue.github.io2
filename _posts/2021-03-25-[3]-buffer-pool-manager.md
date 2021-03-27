---
layout: single
categories: 
    - Database
author: Yue Yin

toc: true
toc_sticky: true
---

## TLDR

```
+--------------------+
| Query Planning     |
+--------------------+
| Operator Execution |
+--------------------+
|   Access Methods   |
+--------------------+
|Buffer Pool Manager | <-
+--------------------+
|   Disk Manager     |
+--------------------+
```

## Buffer Pool Organization

The *buffer pool* is an in-memory cache of pages read from disk. It's a region of memory organized as an array of fixed size pages. Each array entry is called a *frame*. When the DBMS requests a page, an exact copy is placed into one of the frames. 

States maintained by the buffer pool manager:

- **Page Table**: in-memory hash table that maps `page_id` to `frame_id`.
- **Dirty-flag**: set by the thread when it modifies a page. This indicates if a frame has been modified.
- **Pin Counter**: Tracks the number of threads currently reading/writing the page. A thread increments the counter before it accesses the page, and decrease the counter when it finishes accessing it. If a page's pin counter is positive, the buffer pool manager must NOT evict that page from memory. 

Optimizations:

- **Multiple Buffer Pools**: multiple buffer pools for different purposes. This reduces latch contention and improvs locality.
- **Pre-Fetching**: pre-fetch pages based on query plan. Typically used when doing sequential access.
- **Scan Sharing**: multiple query cursors can scan pages together. 

## Replacement Policies

Common:
- LRU
- CLOCK: an approximation of LRU without needing a timestamp for each page

## Implementation

### Latch and Lock

When discussing how a DBMS protects its internal invariants, we make a distinction between locks and latches:

|                      | Locks                        | Latches                   |
| -------------------- | ---------------------------- | ------------------------- |
| What's separated?    | User transactions            | Threads                   |
| What's protected?    | Database's logical contents  | In-memory data structures |
| How long is it held? | Throughout the transaction   | Critical sections         |
| Modes                | Shared, Exclusive, Intention | Read, Write               |
| Deadlock             | Detection/Avoidance          | Avoidance                 |
| ... by ...           | Waits-for, Timeout, Aborts   | Coding discipline         |
| Managed by           | Lock manager                 | Internal data structures  |

### Replacer

The replacer should expose three APIs:

```c++
class Replacer {
 public:
  /* Remove the victim frame as defined by the replacement policy. */
  virtual bool Victim(frame_id_t *frame_id) = 0;

  /* Pins a frame, indicating that it should not be victimized until it is unpinned. */
  virtual void Pin(frame_id_t frame_id) = 0;

  /* Unpins a frame, indicating that it can now be victimized. */
  virtual void Unpin(frame_id_t frame_id) = 0;
}

class LRUReplacer: public Replacer {
 private:
  int capacity;
  std::mutex latch;
  std::list<frame_id_t> evictOrder;
  std::unordered_map<frame_id_t, std::list<frame_id_t>::iterator> evictMap;
}
```

### Buffer Pool Manager

The buffer pool manager exposes 5 APIs. 

```c++
class BufferPoolManager {
 public:
  /* Fetch requested page from the buffer pool. */
  Page *FetchPage(page_id_t page_id) {
    // 1.   Search page table for requested page P;
    // 1.1  If P exists, pin it and return;
    // 1.2  Otherwise, find a replacement page R from the free list or the replacer;
    // 2.   If R is dirty, flush it to disk;
    // 3.   Delete R from the page table and insert P;
    // 4.   Read P from the disk into the buffer pool, and return pointer to P.
  }
    
  /* Unpin the page from the buffer pool. */
  bool UnpinPage(page_id_t page_id, bool is_dirty) {
    // 1.  If the requested page P is not in buffer pool, immediately return;
    // 2.  If is_dirty=true, mark the page as dirty;
    // 3.  Decrements the pin count; If pin count becomes 0, call replacer.Unpin(P).
  }
  
  /* Flush the page to disk. */
  bool FlushPage(page_id_t page_id) {
    // 1.  Check the requested P is not in buffer pool, immediately return;
    // 2.  Ask disk manager to flush the page.
  }
    
  /* Creates a new page in the buffer pool. */
  Page *NewPage(page_id_t *page_id) {
    // 1.  Ask disk manager to allocate a new page P;
    // 2.  If all pages in the buffer pool is pinned, return nullptr;
    // 3.  Pick a victim page V from the free list or the replacer. Evict V;
    // 4.  Zero out V's correspoinding frame in memory, and add P to page table.
  }
    
  /* Deletes a page from the buffer pool. */
  bool DeletePage(page_id_t page_id) {
    // 1.   Ask disk manager to deallocate the page;
    // 2.   Search the page table for the requestd page P;
    // 2.1  If P doesn't exist, return true;
    // 2.2  If P exists, but has positive pin-count, return false;
    // 2.3  Otherwise, remove P from the page table. Add its frame to the free list.
  }
    
 private:
  std::mutex latch;
  size_t pool_size_;
  Page *pages_;
  DiskManager *disk_manager_;
  std::unordered_map<page_id_t, frame_id_t> page_table_; // frame_id indexes into pages_
  Replacer *replacer;
  std::list<frame_id_t> free_list;
}
```

