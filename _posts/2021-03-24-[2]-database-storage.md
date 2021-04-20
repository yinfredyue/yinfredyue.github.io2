---
layout: single
categories: 
    - Database
author: Yue Yin

toc: true
toc_sticky: true
---

## TLDR

This explains how a DBMS represents the database on disk.

How to store a database: disk-oriented DBMS implements buffer pool manager and storage manager.

How to track pages: heap file.

How to store a page: slotted page/log structured.

How to store a tuple: header + data.

How to represent values?

How to represent meda-data about the database: system catalog.

How to choose storage models for different workloads?


## Database Architecture
```
+--------------------+
| Query Planning     |
+--------------------+
| Operator Execution |
+--------------------+
|   Access Methods   |
+--------------------+
|Buffer Pool Manager |
+--------------------+
|   Disk Manager     | <- 
+--------------------+
```

## Storage 

The storage hierarchy:

- Register
- Cache
- Memory
- Disk

We will be talking about **disk-oriented** DBMS. 

## Memory Management

The entire database is stored on disk, organized into pages. To operate on the data, the DBMS needs to bring the data into memory, using a *buffer pool* that manages the data movement back and forth between disk and memory. 

The DBMS should support databases that exceed the amount of physical memory available. The high-level design is similar to virtual memory. 

One way to achieve this virtual memory feature is to use `mmap` and let OS manages the pages. However, because the DBMS has more information about what itself is trying to do, it can manages the memory better than OS. 

Conclusion: disk-oriented DBMS needs to implement a *buffer pool manager*, for better control and performance. 

## File Storage

A DBMS stores a database as files on disk. It may use a file hierarchy or a single file.

The OS doesn't understand the contents of the files. The DBMS's *storage manager* is responsible for managing its files (encoding/decoding/free space management, etc). 

The DBMS organizes the database across one or more files in fixed-size blocks, called *pages*. Pages can store different kinds of data (tuples, indexes, schemas, etc). Usually, the DBMS will not mix these types within a single page, and the page is *self-contained*.

Each page is given a unique identifier. The storage device guarantees an atomic write of the size of the *hardware page*. If the database page is larger than the hardware page, the DBMS needs to take extra measurement to ensure there's no partial write.

## Database Heap

A *heap file*  (the "heap" here has nothing to do with the one in "heap & stack") is a collection of tuples, stored in random order on an unordered collection of pages. Two ways to locate a page on disk given a `page_id`:

- **Linked list**: header page stores a pointer to a list of free pages, and a pointer to data pages. A sequential scan is needed to find a specific page.

- **Page directory**: the DBMS maintains special pages that track locations of data pages and free pages. 

## Page Layout

Each page contains a header that records meta-data about the page's content. Example: page size, checksum, DBMS version, transaction visibility, etc. 

The strawman approach is track the number of tuples stored in a page (using `num_tuples * tuple_size` to calculate page offset), and append to the end to add a new tuple. This is problematic:

- This doesn't work with variable-size tuples, as you cannot calculate the offset using tuple size.
- If we blindly appends to the end, then after each delete, any tuples past the deleted one must be shifted forward. 

There are two main approaches to laying out data in pages:

- **slotted pages**: the *slot array* maps `slot_id` to tuples' page offsets. The tuple array and the slot array grows in opposite direction. 

    This supports variable-length tuple. After deletion, you can do compaction to reduce fragmentations.

    <img src="{{ site.url }}/assets/images/slotted_page.png" alt="slotted_page" width="200"/>

- **log structured**: instead of storing tuples, the DBMS only stores logs about how the database is modified (insert, update, delete). Writes are fast because it just requires one log append; Read requires scan the logs backwards to reconstruct the tuple. Log compaction is usually used.

## Tuple Layout

- **Tuple header**: contains meta-data about the tuple. Example: visibility, bitmap for `NULL` attributes. Note that schema information is not stored. 

- **Tuple data**: actual data for attributes.

- **Unique identifier**: each tuple is assigned a unique identifer within the database. Commonly, `page_id + (slot or offset)` is used. The application should NOT reply on these ids, as the database is free to move pages, or move tuples within a page.

- **Denormalized tuple data**: if two tables are related, the DBMS can "pre-join" tham. The makes read faster but writes more expensive.

## Value Representation

How the DBMS represent integers, floating point numbers, strings/binaries, time/date internally. 

## System Catalog

A *system catalog* stores meta-data about the database: tables, indexes, views, users, permissions, statistics, etc. Usually stored inside the database itself.

## Workloads

- **OLTP**: On-Line Transaction Processing. Usually simple queries that read/update a small amount of related data.
- **OLAP**: On-Line Analytical Processing. Usually complex queries that read large portions of the database. 
- **HTAP**: Hybrid Transaction/Analytical Processing. The combination of both.

## Storage Models

The DBMS can store tuples in different ways to better serve OLTP/OLAP workloads.

- **N-ary Storage Model (NSM)**: also known as "row store". All attributes of a single tuple are stored contiguously. 

    There are two ways to organize a NSM database:

    - Heap-organized tables: tuples are stored in the heap file, which doesn't necessarily define an order.
    - Index-organized tables: tuples are stored in the primary key index itself. 

- **Decomposition Storage Model (DSM)**: also known as "column-store". The DBMS stores a single attribute for all tuples contiguously.

    To put the tuples back together when using a column store, we can use:

    - Fixed-length offsets: padd the values of a field so that all values are of the same length. 
    - Embedded tuple IDs: for each attribute, store the tuple ID with it. 

    Most DBMSs ues fixed-length offsets.

Row stores are usually better for OLTP. Column stores are usually better for OLAP.

