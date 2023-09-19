---
title: "The elegant consensus algorithm - Multi-Paxos - in Java gRPC"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "Distributed System"
    - "Algorithms"
    - "gRPC"
---

## Introduction
Paxos is a distributed consensus algorithm developed by Lamport. It is proved optimal and many systems are built based on it like chubby and zookeeper.

But this article is not going to discuss Lamport's orginal paper but focus on the engineering implementations. My colleague highly recommended Ongaro's lecture of Paxos and said it is the best source of learning Paxos. I cannot agree more after studying it. That being said, I will briefly talk about Paxos and Multi-Paxos and dive into the engineering implementations.

## Multi-Paxos & Paxos
In short, Multi-Paxos is simply the multiple use of Paxos. Paxos is developed by Lamport in 1998 for the distributed consensus problem. There are three agents: proposer, acceptor and listener in Paxos. They can be the same server or separate server depending on the design. In Ongaro's lecture, the listener is part of acceptor thus it ends up with two agents proposer and acceptor (In Lamport's Paxos Made Simple, a server could be proposer, acceptor and lisenter at the same time). To chose an value, proposer is using a 2-phase protocal to lock down an entry in the acceptors' logs.

![](https://i.imgur.com/Tpn7vlY.png)
<center>Figure 1. Protocal of Basci Paxos (from Ontario's lecture)</center>

With Paxos, we can meet the requirements of safety and liveness:
- Safety: Nothing bad is going to happen - at least one value is chosen
- Liveness: Eventually a good thing is going to happen - one value is going to be chosen eventually
	- To avoid livelock, we could use exponential backoff or leader election in Muliti-Paxos

```ad-note
Only proposer knows which value has been chosen, other proposers must execute Paxos with their own proposal to get the chosen value 
```

As said in the beginning, Multi-Paxos is the multiple use of Paxos which means we will use Paxos to choose a value for a log entry every time the client make request. Thus Dr. Ongaro pointed out several issues of Multi-Paxos and suggested solutions:
- Which log entry to use for a given client request?
	- ![](https://i.imgur.com/Sfh9ZY8.png)
	- When `jmp` command is sent to the server `s1`, `s1` need to track the variable `firstUnchosenIndex`
	- If the `firstUnchosenIndex` already has a value, `s1` will send the proposal using the existing value and increment `firstUnchosenIndex` once the value is chosen
	- Then `s1` will start from the beginning until the `firstUnchosenIndex` has no value then `s1` makes proposal from `jmp` command
	- The server could handle the client requests concurrently
		- Suppose we have multiple requests
		- We can try each unchosen index for one independent request
		- So we would need a collection of `UnchosenIndex`
	- Then the application of log entry should be sequential in the state trasaction machine.
- Paxos is slow because of 2-phase protocal and could have intense competition with several proposers (livelock)
	- We should use leader-election to ensure a single Proposer at any given time
		- Bully algorithm - the server with the largest ID is the leader
			- Each server sends the heartbeat to other servers every `T` ms
			- If a server hasn't recevied the heartbeat from server with higher ID in last `2T` ms, it is a leader
				- Accept requests from clients
				- Acts as proposer and acceptor
			- The rest servers are followers
				- Redirected request to leader
				- Acts as acceptor
	- We could send proposal msg to the entire log thus all the log entries would have a global proposal number which can block all old proposals
		- Then we should collect `highestProposal` accepcted for current entry and `noMoreAccepted` bool from the propose request
			- `noMoreAccepted`: no more accepcted values after the current entry
			- If the leader recevies `noMoreAccepted` from the majority acceptors, the leader doesn't have to send any prepare requests and only need accept requests
- How to ensure the full replication across servers?
	- Left issues:
		- Log entries not fully replicated
		- Only proposer knows when entry is chosen
	- Solutions:
		- Keep send accept request until all acceptors respond 
		- mark entires chosen infinite to prevent overwritten
		- each server should track `firstUnchosenIndex`
			- proposer include `firstUnchosenIndex` in its accept requests
			- acceptor should mark all entries chosen if 
				- the entires are lower than `firstUnchosenIndex` 
				- and the entries have the same proposal number
		- acceptor could consult with the leader about unknown index
			- acceptor includes its `firstUnchosenIndex` in accept replies
			- if proposer's `fristUnchosenIndex` is larger then it sends the success request to acceptor to confirm the chosen status
				- and acceptor keep returning the `firstUnchosenIndex` till it catches up
- How to deal with fail-stop issues from client perspective?
	- The client could submit the same request multiple times thus we need idempotent key  associated with each request 
- How to update the configuration of servers?
	- Configuration change
	- impacts
		- the number of servers determines `majority`
		- quorum size
	- $\alpha$ parameter
		- save the configuration as normal log entries
			- configuration is treated like any other CURD commands
		- save the configuration at the entry i
		- then all the entry i + $\alpha$ will use the configuration at entry i if there is configuration file/command
			- e.g. if we updated the configuration, the configuration file will be stored at index $i$ based on paxos
			- and everytime the leader should check the position $i$ to see if there is a configuration file
				- if yes, the configuration should be updated
				- otherwise, the existing configuration should be used
		- if $\alpha$ is too small, the system wind up as a synchronous system
		- if $\alpha$ is too large, the update of configuration could take a while
			- we could use no-op command to quickly update the log entires between $i$ and $i+\alpha$ so configuration could update

## Engineering Impl

This section will summaries the details of the RPC between proposer and acceptors.

### Server
- Each server has its own logs to maintain and we need make sure the logs are consistent across all servers
- Leader election: only leader acts as Proposer to replicate the log entries
- State
	- log
		- value
		- proposal
	- mark the entries known to be chosen
	- each server maintains `firstUnchosenIndex`
- Message
- proposer -> acceptor
		- prepare msg
			- global proposal number
			- index
			- value
		- accept msg
			- for acceptor sent `noMoreAccepted`, only need send accept msg
			- once the majority acceptor sent noMoreAccepted, only need send accept msg
			- keep retrying until all acceptors respond
			- `{proposal: p; index: i; value: v; firstUnchosenIndex: u_i;}`
				- mark all entries before firstUnchoseIndex equal to proposal as chosen
				- put the proposal and value at index i of log 
- acceptor -> proposer
	- response msg
		- `noMoreAccepted`
		- `firstUnchosenIndex`
			- proposer will compare its `firstUnchosenIndex` with accpetor's `firstUnchosenIndex`
				- if >: the propser will send Success msg to acceptor to confirm the entry is took
					- `{index: i; value: v}`
					- then acceptor will updated `acceptedValue[i]` and `acceptedProposal[i]`
					- and return another response contains the updated firstUnchosenIndex

### Client
Any possible command like CRUD operations on database.

### gRPC Service
- Proposer
	- Prepare Msg
		- Proposal
	- Accept Msg
		- Proposal
		- Index
		- Value
		- FirstUnchosenIndex
	- Success Msg
		- Index
		- Value
- Acceptor
	- Response Msg
		- FirstUnchosenIndex
		- noMoreUnaccepted
- Client
	- Command like CRUD

The code can be found at [here]([ydeng11/Multi-Paxos (github.com)](https://github.com/ydeng11/Multi-Paxos)). 