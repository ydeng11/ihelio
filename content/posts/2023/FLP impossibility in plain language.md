---
title: "FLP impossibility in plain language"
date: 2023-09-16T16:17:15-04:00
draft:
categories: 
    - "Programming"
tags: 
    - "Distributed Systems"
---

## Overview
FLP impossibility is to prove there is no algorithm can really achieve totally correct consensus in asychronous system under assumption at most one process is faulty. The paper is very famous and also difficult to understand given the wording. This article is to explain FLP impossibility in a plain way.

## Aschronous System
In FLP paper, there are some settings/assumptions made to describe an aschronous system which is used in the proof. This system has:
- *Consensue Protocal* which contains`N` *processes* with initial value 0 or 1
	- Total correctness
		- at most one process is faulty
		- all values are eventually delivered to non-faulty process 
		- always reach a decision in all runs
	- Partial correctness
		-  No configuration reachable from an initial configuration has more than one decision value
		- For $v \in {0,1}$, some configuration reachable from an initial configuration has decision value $v$  
- A *mesage buffer* which can be seen as a global "key-value store" (a multiset supporting the duplicate write of the same key)
	- Suppose we have a process `p`, there are two functions `p` could use
		- `send(p, m)` - save the pair of `p` and `m` to the message buffer 
		- `receive(p)` - get the value of `p` from the message buffer **but this is non-deterministic** as the message buffer could return null value even the `entry<p, m>` exists - **this is to simulate the aschronous system could suffer from network partition or fail-stop**
			- We could call `receive(p)` infinitely to eventually get the message $m$ delivered though with some delay
- *Configuration* is the essentially the state of this system which is comprised of all processes with initial value and the message buffer in the system
	- To change the configuration a.k.a. state we need `receive(p)` thus some processors could receive new values - this is also called *step/event* in FLP paper stating a process $p$ received a message $m$ 
	- A infinite sequence of *steps/events* are called *run/schedule* which defined a particular execution for a configuration
	- **Note: The FLP paper use run and schedule in different places but I think they are actually interchangeble thus I will use step and run in the following content.**
	- A configuration is decisive if there is only one reachable state from it, otherwise we call this configuration indecisive given it could have decisions either 0 or 1 
- There are two more definitions about *run*
	- Admission run: at most one process is faulty and all messages could still be delivered to non-faulty processes
	- Deciding run: some processes in the run eventually make decisions
	- Non-faulty process could take infinitely many steps in a run and otherwise is considered faulty - the faulty process will only take finite steps since it ceases to process
	- **When every admission run is deciding run, the consencus protocal is totally correct**

To ensure the consencus, we need assure **TAV** a.k.a consensus after every possible execution:
- Termination: All non-fauty processes eventually decide on a value
- Agreement: All deciding processes make the same decision
- Validity: The decision must be propsoed by some processes

## Proof
In this section, I prefer to use the word decisive and indecisive to indicate the state of configuration instead of univalent and bivalent used in the FLP paper.

### Lemma 1
	Suppose there are two disjoint runs, r1 and r2, and an initial configuration C0, the reachable configuration C1 resulted from C0 -> r1 -> r2 is the same as the configuration C2 resulted from C0 -> r2 -> r1.

This lemma is obvious based on the system definition. And this is mainly used to prove lemma 3.
	
### Lemma 2
	The initial configuration in the consencus protocal has indecisive value

The paper make a assumption that the initial configuration should not have indecisive value and find a contradication:
- Say we have $N$ processes, we could have $2^N$ configurations and they could have 0 and 1 as their decisive value
- Then we make a chain of them and make sure the adjacent configurations differ with each other at most 1 like below
	- $\dots,C_0,C_0,C_1,C_1,\dots$
- Thus the only possible cause is the initial value of some process in $C_1$ or $C_0$ and according to the definition of *admissible run* that particualr process, say $p$, could crash
- With the help of admissible deciding run, we could crash $p$ thus it will not be part of the system and we could have the identical *run* for $C_1$ and $C_0$ to eventuall get the same decision value
- **However, this would contradict the assumption because $C_0$ would be indecisive if the final decision value is 1 and otherwise $C_1$ would be indecisive if the final decision value is 0.** 
It is important to note that we call the configuration indecisive when the decision value reachable from them is indecisive - could be 0 or 1. In this proof, it is exactly what we observe as the crash of $p$ affects the decision value of the initial configuration which in turn makes them indecisive.

### Lemma 3
	The indecisive configuration will lead to indecisive configuration

To prove lemma 3, we assume the indecisive configuration will not lead to indecisive configuration. And we need have some assumptions:
- $C$ is the initial configuration which is indecisive
- Suppose we have a step $s$,  $M$ contains all configurations reachable from $C$ without taking step $s$ and $N$ contains all configurations reachable from $C$ with take step $s$.

If configuration $E0$ is in $M$ and we assume configuration $F0$ is rechable from $E0$ by applying step $s$. And $F0$ is in $N$. If $E0$ is not in M and we assume $E0$ is reachable from $F0$ with some steps ($E0$ and $F0$ indicates their decisive value is 0).

Bascially we are clarifying that all configuration reachable from $C$ could have one and only one decisive value which should be 0 here since $C$ is assumed to have 0 in this case. Similarly we could conclude that $C$ could lead to configuration whose decisive value is 1 if $C$ is 1. That being said, **we could have configurations with either 0 or 1 in $M$ and $N$, but no configuration should be indecisive in the two sets.**

<p align="center">
	<img src="https://i.imgur.com/mdd1Cxz.png"/>
</p>
<center>Figure 1. The possible destination from indecisive configuration C when its value is 0 </center>

Then we could continue our proof which is the **most important part** in the FLP paper. Given we know there are two possible configuration differ with each other at 1 value, let's call them configuration $D0$ and $D1$ (each with decisive value 0 and 1). We know there is one step $s'$ with process $p$ causing the difference between $D0$ and $D1$. And we assume there is another step $s''$ with process $p''$ (the message in these steps are different) . 
- When $p\neq p''$, we know $D0$ could move to $D1$ by applying step $s'$ and further move to $E1$ by applying step $s''$ . Because $s' \neq s''$ (different process or different message or both), we know $D1$ won't go to $E0$. Hence we know $D0$ will go to $E0$ by just applying step $s''$. However, after we apply step $s'$ to $E0$, $E0$ should go to $E1$ according to lemma 1, **which contradicts our assumption as some configuration become indecisive by just changing the order of steps**.

<center><img src="https://i.imgur.com/wtR4nYr.png
"></center>
<center>Figure 2. D0 will actually lead to indecisive configuration</center>

- When $p=p''$, this process could crash and take no steps given we are allowing one faulty process and therefore we have a run $R$ without $p$. Then we could have 3 possibilities:
	- $D0$ goest to configuration $A$ by applying the run $R$
	- $D0$ goes to $D1$ with step $s'$, and goes to $E1$ with step $s''$, and goes to $F1$ with run $R$
		- The decisive value in this path stays as 1
		- Note: step $s'$ still differ with $s''$ because they refer to different messages
	- $D0$ goes to $E0$ with step $s''$ and goes to $F0$ with run $R$
		- The decisive value in this path stays as 0
And given messages could be delayed by not complete lost, we could still:
- take step $s'$ at configuration $A$ to move to $F0$
- or take step $s'$ and step $s''$ at configuration $A$ to move to $F1$
**This contradicts the assumption as A is indecive now since it can reach both 0 and 1**.

<center><img src="https://i.imgur.com/gkpaveh.png"></center>
<center>Figure 3. The D0 will get to another configuration which becomes indecisive with one faulty process</center>

## Conclusion
The FLP is proving the impossibility of totally correct concensus in a weak form of asynchronous system and let alone a stronger form of asyhchronous system. But this paper also indicates the research direction to resolve distributed concensus probelm like:
- relaxing the restrictions like using leader election which makes a weaker form of asynchronous system but provide optimal concensus
- using timeout to make sure the no process could infinitely try to get messages
- using failure detectors to inform the system failures and take corresponding actions 

---
# References
1. Fischer, M. J., Lynch, N. A., & Paterson, M. S. (1985). Impossibility of distributed consensus with one faulty process. _Journal of the ACM (JACM)_, _32_(2), 374-382.
2. [A Brief Tour of FLP Impossibility](https://www.the-paper-trail.org/post/2008-08-13-a-brief-tour-of-flp-impossibility/)
3. [John Feminella on Impossibility of Distributed Consensus with One Faulty Process](https://www.youtube.com/watch?v=Vmlj-67aymw) 

