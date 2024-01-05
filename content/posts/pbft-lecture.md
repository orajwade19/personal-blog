+++
title = 'Notes on PBFT Lecture by Barbara Liskov'
date = 2024-01-05T12:10:07-05:00
+++

# [Lecture link](https://www.youtube.com/watch?v=Uj638eFIWg8)
# Notes from BFT Talk by Barbara Liskov

- In the 80s, the focus was on full-stop failures due to:
  1. Hardware unreliability.
  2. The perceived difficulty of the problem at that time.

## Failstop

- We require 2f + 1 replicas for totally ordered transactions.
- Two-phase protocol (s)
  1. First phase determines what happened.
  2. Second phase reflects the actual changes.

## Byzantine

- Today, Byzantine failure tolerance requires 3f + 1 replicas.
- Three phases involved.
- PBFT provides SMR (State Machine Replication).
- Prevents inconsistent state.
- Clients can still write "bad" data, but access control is employed to prevent this.
- Execution of some requests occurs in the same order.

**Note:**
- Non-determinism can interfere; for example, the last read time in a filesystem.
- The need to convert this into a deterministic process.
