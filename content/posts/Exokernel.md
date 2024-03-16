+++ title = 'Exokernel' date = 2024-03-16T12:10:07-05:00 +++
# Intro
Traditional OS introduces concepts like files, memory, IPC, processes, hiding real machine information. This approach is criticized for three reasons:
1. No domain-specific optimization possible
2. Discourages changes to current abstractions
3. Hard to add new abstractions

## How does the paper solve this?
The paper proposes delegating these abstractions to the application level. The kernel implements only hardware multiplexing and separates protection from management. The Exokernel has three building blocks:
1. **Secure Bindings:** Apps securely use machine resources.
2. **Visible Resource Revocation:** Apps participate in a resource revocation protocol.
3. **Abort Protocol:** To deal with apps that don't respect "2."

In contrast to Microkernels, Exokernel systems implement even virtual memory and IPC at the user (app) level, not in the kernel.

# Motivation
1. Worse performance due to potentially unnecessary abstractions. Example: Threads, relational DBs hurt by LRU caching strategy.

> **My understanding so far:
With Exokernel, for example, DBs can implement "tables" without having to deal with files.**

# Design
1. Expose physical hardware securely: Provide protection on top of hardware access (allocation, tracking ownership, revocation).
2. Basically, the library OS should have all the information about physical attributes, i.e., HDD arm position, physical address of memory page, etc. Resource policy is handled at the application level; Exokernel just handles allocation and revocation.

### Secure Bindings
1. separate authorization from use. only check permissions at bind time. reduces overhead. analogy: Car keys and car rental contract.\
This is implemented in the paper with \
hardware support: memory mapping from virtual to physical is bind time,actual use of page is access \
dynamic code generation : packet filtering using predicates


> **The word "capability" in the paper seems to refer to access permissions.**

2. Exokernel records "capabilities" specified by the library OS for memory pages. checks them when needed.

### Visible Resource Revocation
The Kernel requests that the library OS return previously allocated/securely bound resources. this allows the library OS to save, for example, memory content to disk. This is a dialogue between kernel and OS. Abort protocol works differently.


### Abort protocol
in the paper, instead of killing the process, the kernel revokes ownership of resources and gives the library OS a reposession vector. library OS can then act accordingly. 

# Evaluation
Aegis + ExOS is compared with ULTRIX, running on DEC machines.

# Implementation
## Aegis, the Kernel
### CPU
1. Aegis represents CPU as a linear vector. each element of the vector corresponds to a time slice. position in the vector estabilishes a bound for "when" an app will run.
2. Fairness is achieved by using time taken by an app to save context. More time taken to save context -> less time for execution.
3. Aegis does have guaranteed mappings which can be used by library OS to bootstrap some code and data. TLB misses are categorized into either guaranteed mappings or user segment misses. in case of second, control is relegated back to app. in case of first, aegis handles the exception.
