+++
title = "Memberlist: A Building Block for Distributed Systems"
date = "2025-06-23"
author = "Son Vu Thai"
tags = [
    "Distributed System",
]
+++

# What is membership protocol

> A group membership protocol ensures agreement and consistent commit actions among group members to maintain a sequence of identical group views in spite of continuous changes, either voluntary or otherwise, in processors' membership status.

In a distributed system with multiple processes, the process group dynamically updates its membership in response to specific events: a process is removed upon failure, rejoined upon recovery, added when a new process joins, or removed when a process leaves. Group members query the current membership view to inform their actions and are able to take actions. Agreement on the membership of a group of processes is a must to avoid inconsistency problems. There are several aproach to disscus into detail below.

1. Attendance List Protocol
   Developed by Flaviu Cristian, In this protocol, the membership is checked by sending a datagram to all the members some time after a join is completed. This datagram reaches all members within a bounded time and all members check to see if they received it within the right time. If there is a failure, at least one of them does not receive the list and it issues a new join phase. In this phase, the member which has failed does not participate and his membership is removed from the group by other members. This protocol has a reduced message overhead in the absence of changes and is more efficient than the periodic broadcast protocol. This reduced overhead leads to an increase in the failure detection time
2. Periodic Broadcast Protocol
