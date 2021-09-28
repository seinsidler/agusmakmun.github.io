---
layout: post
title: Building a Blockchain (in Go) for Fun (Pt. 2)
date: 2021-09-16 21:11:27
categories: [crytocurrency, golang]
---
## The DHT

Where we last left of, we setup a nice little local blockchain that we can run on our local network, using a proof of work consensus for posting blocks. Not bad for one hefty blog post! But we still have a long hill to climb ahead of us. And one of the problems we encountered last time was that each node had to be given an address of another node in order to connect to the network. Wouldn't it be nice if we could just have our node connect to the network on its own?

That's where a DHT, or a *Distributed Hash Table*, comes into play. The IPFS documentation (written by the same people who created the library we are using to build the network) says this about DHTs:
>A distributed hash table (DHT) is a distributed system for mapping keys to values. In IPFS, the DHT is used as the fundamental component of the content routing system and acts like a cross between a catalog and a navigation system. It maps what the user is looking for to the peer that is storing the matching content. Think of it as a huge table that stores who has what data.

We can use this distributed table to store address information of other nodes on the network. But how does this information from the table get sent to us? Now we must look into the  Kademlia algorithm! 

## The Kademlia Algorithm
The DHT is just a data structure, but there are many ways to read, write, and store data in a DHT. After all, a DHT is stored on many nodes (ideally) and how this information is sent and the DHT is implemented is entirely up to the programmer. The Kademlia algorithm is an attempt to create a DHT and a protocol in which information is stored and retrieved on the network. I'm planning on writing another blog post on the Kademlia algorithm in the future, but I'll just stick to the very basics here. If you are curious to read more about it right now, I highly recommend reading more about it [here](https://docs.ipfs.io/concepts/dht/#kademlia) on the IPFS documentation page or the original paper [here](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf). 

Essentially, the Kademlia algorithm stores information on each node based on the distance away each node's ID is from itself using XOR as a metric of distance. *What does that even mean?* Well, with the libp2p library and on IPFS are hash values, meaning they come from a function that is hard to invert and the output is of a fixed size. To find the distance between two nodes in the Kademlia algorithm, we use the XOR operation between the address. XOR means that for every bit (a bit is 0 or 1) in the address, we compare if they are the same or not. If they are different, we write a 1, and if they are the same, we write a zero. And example would be 
```
1110 XOR 0101 = 1011
```

This is a pretty odd thing to use as metric for distance, a node with an address that is one off by this metric could be on the other side of the world! But mathematicians have a much looser definition of what you can measure distance with. While physical distance in feet and yards is pretty nice to measure the distance between cities, what if we wanted to measure what is the shortest chain of friend-of-a-friends I have to get to Kevin Bacon (the classic six degrees of Kevin Bacon)? While this certainly is a measure of relative distance that we might want to know, it certainly wouldn't help to use yards for this measurement! But I digress...

Kademlia stores information about our node addresses by putting them into buckets relative to how far away they are using our fancy XOR metric. Whenever any node receives a message from another node, the receiving node updates the DHT with the address information from the other node. This helps any node keep an updated list of live nodes, all sorted based on a distance away in the address space. When there are no nodes on the network, often times a bootstrap node is given.

That's all I'll say on the algorithm for now because we don't interact with it too much under the hood for now. But I will cover the algorithm more in depth in a future blog post as I find this algorithm pretty intriguing! 

## 