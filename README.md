# Mechanism Description 

Our key-value store system enforces causal consistency by meticulously tracking causal dependencies. This is achieved through the exchange of causal metadata in requests and responses between clients and replicas. This section details the system's approach to manage causal dependencies.

#### Causal Metadata Handling

- **Initial Interaction**: When a client first interacts with the key-value store, it sends a request with the `causal-metadata` field set to `null`. This indicates the beginning of its interaction with the store.
  
- **Metadata Propagation**: Each client request and corresponding response includes a `causal-metadata` field. Clients don't interpret the value of this field but persistently pass the latest received `causal-metadata` in their subsequent requests.

- **Vector Clocks**: Our system primarily utilizes vector clocks to maintain causal relationships. Each replica and client maintains its own vector clock. The vector clock is a critical component of the `causal-metadata`.

#### Vector Clock Operations

- **Increment on Updates (Rule One)**: Every time a replica performs an update (PUT or DELETE operations), its entry in the vector clock is incremented. This increment reflects the occurrence of a new event (update) at that replica.

- **Merging Clocks (Rule Two)**: Upon receiving a request with `causal-metadata`, a replica merges its vector clock with the one received. It updates its clock entries to the maximum values between its own clock and the received clock. This process ensures the replica acknowledges the latest known states of other replicas.

- **Causality Check**: is_causal(meta_clock): Before processing read (GET) requests, the system checks if the requesting client's causal history (based on its `causal-metadata`) is consistent with the replica's state. If the replica's state is causally ahead, the request is deferred, ensuring read operations respect causal order.

#### System Design Highlights

- **Replica Management**: check_replica_health():, The system dynamically manages replicas, adding or removing them based on their health status. This flexibility ensures robust performance and fault tolerance.

- **Data Synchronization**: sync(replica): Replicas periodically synchronize data and vector clocks with each other, ensuring that all replicas maintain a causally consistent state of the key-value store.

- **Broadcast Updates**: Updates from one replica are broadcasted to other replicas to maintain a globally consistent state. This includes both the updated key-value pairs and the updated vector clock. A similar approach is used for checking docker containers that are in the view using add_replica_from_view(replica) and remove_replica_from_view(replica):

- **Error Handling**: The system is designed to handle various errors gracefully, ensuring reliability and robustness in operations.

#### Client-Replica Interaction

- **Client Request Format**: `{ ... "causal-metadata": <causal-metadata>, ... }`
- **Replica Response Format**: Includes updated `causal-metadata` post operation.

This design ensures that our key-value store operates in a causally consistent manner, respecting the causal relationships of operations across different replicas and clients.

# Acknowledgements
N/A

# Citations 
https://queue.acm.org/detail.cfm?id=2917756
https://levelup.gitconnected.com/distributed-systems-physical-logical-and-vector-clocks-7ca989f5f780
https://pypi.org/project/requests/
https://docs.python.org/3/library/json.html
https://docs.python.org/3/library/os.html
https://docs.python.org/3/library/logging.html

# Contributions
Brian Bui - developer, debugging, code review
Monyreak Kit -  developer, debugging, code review


