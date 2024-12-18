+++
title = "Endianness"
description = "Some note about Endianness."
date = "2024-12-17"
author = "Son Vu Thai"
tags = [
    "algorithm",
    "computing",
]
+++

The term *endianness* describes the order in which computer memory stores a sequence of bytes. Little endian means the lower significant bytes get the lower addresses. Big endian means the other way around. 
For example, with value = -12, Little endian, in memory , would be:
```
000000: F4
000001: FF
```
Big endian, in other word, would be:
```
000001: FF
000000: F4
```
Therefore, with different CPU / diffrent hashing algorithm, when converted from bytes to int, the result might be different. Here's an example

-----------------------------------------------------------------------------------------------------------------

**Computer 1:**
```python
import sys
import hashlib
sys.byteorder # little

h = hashlib.md5(b"test")
dig = h.digest() # digest() return bytes sequence, in this case output is in little endian
print(int.from_bytes(dig, "little")) # 327925494462908176265137084817260384009
    
```
-----------------------------------------------------------------------------------------------------------------
**Computer 2:**
```python
import sys
import hashlib
sys.byteorder # big

h = hashlib.md5(b"test")
dig = h.digest() # digest() return bytes sequence, in this case output is in little endian
print(int.from_bytes(dig, "big")) # 12707736894140473154801792860916528374
```


