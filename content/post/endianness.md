+++
title = "Endianness"
description = "Some note about Endianness."
date = "2024-12-17"
author = "Son Vu Thai"
+++

The term *endianness* describes the order in which computer memory stores a sequence of bytes. Little endian means the lower significant bytes get the lower addresses. Big endian means the other way around. 
For example, with value = -12, Little endian, in memory , would be:
```
000000: F4
000001: FF
```
Big endian, in other word, would be:
```
000000: F4
000001: FF
```
Therefore, with different CPU, when converted from bytes to int, the result might be different.
