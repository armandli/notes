### Raft Consensus Algorithm

Building reliable state machine: build replicated state machine, having a replicated log to ensure that each state machine execute the same commands in the same order, and produce the same result. Consensus module ensure proper log replication. System makes progress as long as any majority of servers are up.

RSM with replicated log survives delayed/lost message, fail-stop, but not byzantine.

#### Paxos Single Decree - classic consensus algorithm, one of the many Paxos algorithm
a process to elect one and only one value once and for all from all machines

2 steps process, have a *cluser of propsers* and *cluster of acceptors*.
1. proposers all try to *propose* a proposal with a *proposal number*
2. acceptors receives the proposals, if acceptor *has not accepted any values*, it looks at all existing proposals received, and reply to the poposal with the *highest proposal value greater than* any existing proposal value received, and will *not reply* to any other proposals with lower proposal value; if acceptor has *already accepted a proposal*, the acceptor will *send out the value corresponding to the accepted proposal*
3. the proposer who received enough number of accepted message more than *quorum*, then go on to sent *accept* message; the message contains the proposal number, and if the accepted reply *already contains a value*, use that value to be the result bounce to the proposal, *otherwise the proposer choose its own value* to be bound to the proposal number
4. proposers again check if the *proposal number* is *greater than and equal to* any received proposal number, if so, send back to that proposer *accepted message*

problems:
1. algorithm does not address liveness, if at any time proposer stops sending messages, no agreement is achieved and no result is achieved
2. why first response has to check for highest value greater than, and then second step checks for value greater than equals to
3. algorithm only describes one step of the consensus process for RSM, typically a replicated log will have to create multiple values, the algorithm does not address the full problem
4. no description of how to choose proposal number
5. how to do membership management

#### Raft Decomposition
1. leader election, start off with a leader, choose a new one if previous one crashes
2. log replication, leader accepts command from client, appends to its log, replicates log to other machines
3. safty, keeps log consistent, only machines with up to date log can become leader

there is a performance issue, but consensus is fundamentally non-scalable. it is possible to divide a cluster into multiple smaller clusters, each doing its own thing.

#### Raft Machine States and RPCs
`follower -> candidate -> leader`

* follower converts to candidate if heartbeat was not received within timeout period, then follower state change to candidate
* candidate will send out RPC *RequestVote* to get elected as leader
* if candidate recieves enough votes more than quorum, candidate becomes leader state
* leader issues RPC *AppendEntries* to all followers to replicate logs, and send regular heartbeats to maintain leadership

follower does nothing but to act as replicators of logs

#### Terms
in order to create consensus, need method to detect obsolete information. e.g. detect a leader is no longer a leader. we call this term, as in the term a leader is going to rule.

each term has a number, a *term number*.

each term begins with a election process. it is possible that during leader election, we get split vote, then no leader is elected and another term is started with election process.

Each machine can have different view of the term, due to machine instability. each machine keeps to itself the term number, the server constant exchange this term number. if each machine sees the other machine has a higher term value, the machine updates its term number and immediately becomes a follower. 

eventually all machines will converge on the same term number.

#### Leader Election
1. a follower becomes a candidate when heartbeat timed out. note timeout on each machine could be different due to no complete time synchronization, but partial time synchronization
2. candidate increase its own term number, vote for itself, then send *RequestVote* RPC to other machines
3. multiple possible outcome follows:
* received a majority vote, then candidate becomes leader
* received a RPC heartbeat from leader having the same or higher term number, then becomes follower
* waited for votes, but did not reach quorum and get elected in time, and election timed out, then go to step 2 and redo leader election

#### Election Correctness
* ***safety*** allow at most one leader per term
* ***liveness*** makes sure some candidate must eventually win
* ***randomness*** approach is simpler than ranking, each machine timeout randomly. this is different from Paxos where Paxos is relying on ranking, which includes many special cases, whereas random solution works and is simple

#### Normal Operation
1. client send command to leader
2. leader append command to log
3. leader send *AppendEntries* to all followers
4. once new entry is **committed**, header executes command in its state machine and return result to client, leader notifies all followers of committed entries in subsequent *AppendEntries*, follower executes committed command on their state machine
* for crached followers, leader to keep retrying until they succeed
* this is optimal performance in common case, there is one successful RPC to any majority of servers
* the two step commit is used to ensure replicability

#### Log Structure
each log segment contains 2 things:
* term number, the term number this log segment is created
* the command itself

it can be observed from logs of all machines which segment of the log has been committed, and that is if a segment is replicated to the majority of the machines.

logs on each machine can become inconsistent overtime. this is because the leader log could write to its own log before receiving confirmation from other machines, and then crash to have another machine elected leader, the new leader did not receive the logs pushed from the previous leader, and being writing a new log sequence.

**It is assumed the leader's log is always correct**, normal operation will repair inconsistencies

goal is to keep the highest level of consistency between machines. in order to do that, there are following rules:
* log entries on different servers have same index, there can be no holes within a log on any machine where one segment is missing and then next segment exists
* term order of the log is always sorted, always lower number term is before higher order term.

guarantee ***log matching property***, such that if 2 entries are at the same index position in the log, and the log segment has the same term, then the log segment should be identical

*AppendEntries* RPC include log repair. when the RPC is sent out, it includes the previous log segment's term and data. If the previous entry does not match on the follower, the follower send reject message signalling log difference. the leader then backs up one and try again, as in sending just 2 entries including the current entry, it sends 3 entries, and 4, and 5 and so on

once a log segment is committed, all future leaders must store that entry, machines with incomplete logs must not get elected. candidates include index and term of last log entry in *RequestVote* RPC, and voting machines denies vote if its log is more up to date. logs are ranked by `last term, last index`

#### Adding and Removing Machines
TODO
