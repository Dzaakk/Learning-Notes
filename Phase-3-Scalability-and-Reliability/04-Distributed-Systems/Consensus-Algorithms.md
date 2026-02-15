Consensus Algorithms
===

## What is Consensus?
Consensus is getting multiple nodes to agree on a single value or decision, even when some nodes may fail or the network is unreliable.

**Core Problem:** How can independent computers, communicating only through unreliable networks, collectively make a decision they all agree on?

## Why Consensus is Necessary

### Maintaining Consistency Across Replicas
All replicas must process operations in the same order to stay consistent.\
**Example:** When replicating database writes across 3 servers, all must agree on the order of transactions.

### Leader Election
Nodes must agree on exactly one leader to avoid split-brain scenarios.\
**Example:** Primary database selection in replication cluster.

### Distributed Transactions
All participants must agree to commit or abort together.\
**Example:** Two-phases commit across multiple databases.

### Configuration Management
All nodes must see the same system configuration.\
**Example:** Cluster membership changes, routing rules.

## The FLP Impossibility Result
**Fischer-Lynch-Paterson Theorem:** In an asynchronous distributed system where even one node can fail, no consensus algorithm can guarantee termination in all cases.

**Why?** Cannot distinguish between a failed node and an extremely slow one with certainty.

**Practical Implication:** Real consensus algorithms work around by:
- Using timing assumptions (partial synchrony)
- Accepting rare non-termination cases
- Using randomization

## Paxos
Classic consensus algorithm by Leslie Lamport. Famous for being correct but difficult to understand and implement.

### Roles
- **Proposers:** Suggest values
- **Acceptors:** Vote on proposals
- **Learners:** Learn the chosen value

### Two-Phase Process
**Phase 1: Prepare**
1. Proposer selects unique proposal number N
2. Sends prepare(N) to acceptors
3. Acceptor promises not to accept proposals < N
4. Acceptor returns any previously accepted value

**Phase 2: Accept**
1. If proposer gets majority responses, sends accept(N, value)
2. Value must be from highest-numbered prior proposal, or proposers choice
3. Acceptors accept unless they promised to reject this number
4. If majority accepts, value is chosen

**Key Properties**
- Once a value is chosen, all future proposals must propose the same value
- Guarantees safety (at most one value chosen)
- Requires majority (quorum) to make decisions

**Problems**
- Complex to implement correctly
- Livelock possible when multiple proposers compete
- Abstract and mathematically dense
- Led to many incorrect implementations

## Raft
Designed explicitly to be more understandable than Paxos while providing equivalent guarantees.

### Core Concpets
**Terms:** Time divided into numbered terms, Each term begins with an election.

**States:** Each node is either
- **Leader:** Handles all client requests
- **Follower:** Passive, responds to leader
- **Candidate:** Seeking votes to become leader

**Replicated Log:** Leader appends commands to log, replicates to followers.

### Leader Election
1. Follower's election timeout expires → becomes candidate
2. Candidate increments term, votes for itself, requests votes
3. Other nodes vote if:
    - haven't voted this term yet
    - Candidate's log is at least as up-to-date
4. Candidate with majority votes becomes leader
5. Leader sends heartbeats to prevent new elections

**Split Vote Handling:** Randomized timeouts make split votes unlikely. If occurs, timeout expires and new election starts.

### Log Replication
1. Leader receives command from client
2. Leader appends to its log
3. Leader sends AppendEntries to all followers
4. Followers append and acknowledge
5. Leader commits when majority acknowledges
6. Leader applies to state machine, notifies followers
7. Follower apply committed entries

### Safety Guarantees
- **Election Safety:** At most one leader per term
- **Leader Append-Only:** Leader never overwrites its log
- **Log Matching:** If logs contain same entry at same index, they're identical up to that point
- **Leader Completeness:** If entry committed in a term, it's in all future leader logs
- **State Machine Safety:** If node applies entry at index, no other node applies different entry at that index

### Handling Failures
- **Follower Failure:** Leader tracks next index for each follower, sends missing entries when follower recovers.
- **Leader Failure:** Election timeout triggers new election. New leader's log contains all committed entries (guaranteed by voting rules).

- **Uncommitted Entries:** May be lost if not replicated to majority. Acceptable since never acknowledged to clients.