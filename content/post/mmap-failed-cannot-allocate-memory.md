+++
title = "'mmap failed cannot allocate memory' - Here's why and how to fix it"
date = "2025-04-17"
author = "Son Vu Thai"
tags = [
    "System",
]
+++

When running a high-load performance monitoring platform with [Cortex](https://github.com/cortexproject/cortex). Once day the system crashed, the Ingester ran into CrashBackoffLoop with the following mmap error in log file:

```
opening existing TSDBs:  failed to open TSDB: 10 errors: mmap, size 31301: cannot allocate memory; mmap files: mmap, size 16585: cannot allocate memory; mmap, size 29350: cannot allocate memory; mmap, size 30531: cannot allocate memory; mmap, size 29350: cannot allocate memory; mmap files ....
```

## Scenario where this error pops up

I need to confirm 2 point of views:

- Are there enough free memory?
- Are maximum number of mappings exceeded?

I noticed this error occurred when the Ingester performed disk read/write operations, such as reading the WAL or compacting blocks and writing to disk. The servers had plenty of free memory, and disk space. Therefore, the answer of first question was YES.

Now, let count the number of memory mapping regions by Ingester:

```bash
# cat /proc/1/maps| wc -l
```
Process was crashed when it over `vm.max_map_count`. It was fixed by increase the value of `vm.max_map_count`, so it was exactly the reason.

## But, what is vm.max_map_count

vm.max_map_count is about the number of memory mapping regions a process can have, not their size. Here's how it works:

- Process A can create up to vm.max_map_count separate memory mapping regions
- Each region can be of any size (small or large)
- The total number of regions can't exceed vm.max_map_count
- The total size of all regions is limited by available virtual memory, not by max_map_count

For example:

- If vm.max_map_count is 65530 (default)
- Process A could have 65,530 separate 4KB mappings (many small regions)
- OR it could have 1,000 mappings of 10MB each (fewer, larger regions)
- OR just 10 mappings of 1GB each (very few, very large regions)

As long as:

- The total number of mappings stays below vm.max_map_count
- The total memory used stays within physical RAM + swap limits
- The process stays within its other resource limits

Then yes, the process should run without memory allocation errors. The "cannot allocate memory" error happens when either you hit the mapping count limit or run out of actual memory resources.
