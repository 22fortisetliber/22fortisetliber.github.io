+++
title = "Quick Note about Memory"
date = "2024-12-18"
author = "Son Vu Thai"
tags = [
    "Computing",
    "Virtualization",
]
+++

## 1. Difference between vSphere monitor and Window Task Manager

Windows Task Manager will show memory consumed (i.e. allocated at some point in the past and not released since then). 
This doesn't necessarily mean all of that memory is active; much of it could be inactive. vSphere shows memory demand/active: the amount of guest memory that's been recently accessed. 
This could be very different to what the guest OS thinks it's allocated.

Inside the Guest OS

**Commit (KB)** = ”Amount of virtual memory that is reserved for use by a process.”
In other words, this is the amount of memory that a guest process has reserved, which doesn’t necessarily mean it is actively using it.

**Working Set (KB)** = ”Amount of memory in the private working set plus the amount of memory the process is using that can be shared by other processes.”
This is obviously a pretty vague description, also defined/described as “set of pages in the virtual address space of the process that are currently resident in physical memory”. This second description is MUCH better. 
What this is saying is that the working set is memory from the process that has been assigned a location on the memory. 
Again this isn’t memory necessarily being actively used/accessed, but just memory that has a residence/location. 
An analogy that just came to me(so be prepared)…Think of it as a hotel room that has been assigned to you. 
You have that whole room if you want it, but its possible you may only Really use the room for sleeping. 
Which means even though that location is assigned to you, you aren’t actively using it for a majority of the time you have it. You can be so greedy sometimes!!! 🙂
Similarly some processes like to acquire and hold on to memory even though they are not actively using it.

**Shared (KB)** = Don’t need a quote for this one. Essentially the memory that is being shared with another process

**Private (KB)** = This is the amount of memory that is assigned only to you.

=> Private+Shared=WorkingSet

Now for memory as vCenter views it.

**Granted** = The amount of memory vCenter presents to the VM/Guest. This is the amount of  “physical” memory the guest believes it has.

**Consumed** = ”..the amount of machine memory allocated to the virtual machine, accounting for the savings in shared memory.”

From vSphere Resource Management Guide: 

**Active** = ESX uses a statistical sampling approach to estimate the aggregate virtual machine working set size without any guest involvement. 
At the beginning of each sampling period, the hypervisor intentionally invalidates several randomly selected guest physical pages and starts to monitor the guest accesses to them. 
At the end of the sampling period, the fraction of actively used memory can be estimated
as the fraction of the invalidated pages that are re-accessed by the guest during the epoch. 
ESX uses a statistical sampling approach to estimate the aggregate virtual machine working set size without any guest involvement. 
At the beginning of each sampling period, the hypervisor intentionally invalidates several randomly selected guest physical pages and starts to monitor the guest accesses to them.
At the end of the sampling period, the fraction of actively used memory can be estimated as the fraction of the invalidated pages that are re-accessed by the guest during the epoch.”
In layman’s terms, vCenter and Esx/Esxi have a really complicated algorithm for calculating how much of the memory is actively, yes actively being used by the guest.
