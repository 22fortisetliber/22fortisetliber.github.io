+++
title = "Quick Note about CPU"
date = "2024-12-18"
author = "Son Vu Thai"
tags = [
    "Computing",
    "Virtualization",
]
+++

## CPU Load
CPU load is the number of processes that are using, or want to use, CPU time, or queued up processes ready to use CPU.  This can also be referred to as the run queue length.  Let’s say for example you have 1 CPU with 1 core.  
If you have a load average of 1 that would mean you are at capacity and anything more than that would start to queue.  If you have an average CPU load of .5 that would mean you are at half capacity giving you room for more load. 

## CPU Usage
CPU Usage = CPU_Time_Busy / Total_CPU_Time * 100

## Difference between Hypervisor CPU Usage and VM CPU Usage

The CPU usage from libvirt consists of:

**VCPU Usage:** the physical CPU time consumed by virtual CPU of virtual machine

**Hypervisor:** the physical CPU time consumed by emulator stuffs

Therefore, the time used by the VCPUs to run the virtual machine = sum(vcpu)[1...n]
