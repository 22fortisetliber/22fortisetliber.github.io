+++
title = "Consistent Hashing"
date = "2024-12-17"
author = "Son Vu Thai"
tags = [
    "Algorithm",
]
+++
## Introduction
Consistent hashing was originally designed for distributed systems and later gradually used in load balancing. 

## Concept 
In a distributed system, N keys need to be stored in multiple shard nodes. In order to retrieve a specific key, the client needs to know which node each key is stored in.
The most straightforward method is to maintain a global key index, but this method is too costly. Therefore, there needs to be an algorithm where each node can independently calculate which node the key belongs to without the need to communicate with each other, while still ensuring that the keys are allocated fairly among these nodes.

The simplest method is to use hashing, by which all keys are assigned to a certain number of buckets using hash, and then the client uses the same hash method to retrieve the path. Howerver, the drawback of it is that the number of backend server change cause large portion of traffic shoudld be re-assgined. Therefore, new algorithm is needed, one that can adapt to the dynamic changes of backend nodes and achieve optimal dynamic balance with minimal changes 

Consistent Hashing has 4 basic requirements:

**Balance:** Every node holds a balanced number of keys.

**Monotonicity:** When a new node is added, keys will only be transferred from the old node to the new node. -> to minimize the amount of data involved when nodes change.

**Spread:** When each client can only see a portion of the nodes, the number of nodes that the key belongs to should be as small as possible. -> the consistent hashing algorithm must also ensure that the allocation of the same key is not too dispersed

**Load:** For each client, at most a certain number of keys are considered to be stored in a node -> ensuring that a node does not store too many keys.

### 1. Hash ring
Consistent Hashing is a distributed hashing scheme that operates independently of the number of servers or objects in a distributed hash table
![image](https://github.com/user-attachments/assets/791c8506-93c5-4327-bfd6-c4eac672d4d6)

First, we select the hashing algorithm and sorted all hash result of all nodes into a list. When the request arrived, first, we calculate the hash results, then perform a search in the pre-computed list to find the node that responsible for handling the request. If the hash values of the request has exceeded the maximum values of list, we choose the smallest one.
**Disadvantage:** The distribution of node is not balanced. Adding or removing nodes can cause a large of request should be re-assigned

### 2. Virtual Nodes
Common terms:
. Real server: nodes, servers, shards
. Virtual nodes: vnode, replicas, tokens

Each real node corresponds to N virtual nodes. All virtual nodes are ramdomly distributed on the hash ring. After calculate the hash, we sequentially search for the nearest virtual nodes the query the real node represented by it.

**Advantages:** Very good balance.

**Disadvantages:** Requires storage of bidirectional relationships between all virtual nodes and nodes, requiring a large memory.

### 3. Ketama
Nginx and memcached also use this algorithm, essentially a hash ring algorithm with virtual nodes, although it’s unclear why it needs its own name.

### 4. Spread-minimizing tokens



### 5. Maglev Hashing

