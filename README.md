# Distributed Key-Value Storage System

### 1\. Brief Description
A high-availability, strongly consistent Key-Value storage system built on the Raft Consensus Algorithm to ensure data reliability and fault tolerance across a cluster of nodes.

| Feature | Description |
| :--- | :--- |
| **Consistency** | **Strong (Linearizable) Consistency** for writes, guaranteed by Raft's log replication. |
| **Availability** | Fault-tolerant operation; the system remains operational as long as a **majority** of nodes are active. |
| **Technology** | Implemented in **Java** and uses a custom **RPC layer** for inter-node communication. |

-----

### 2\. System Architecture

The project is structured into three main modules, providing a clear **separation of concerns** between consensus and storage:

#### 1\. `distribute-java-core` (The Consensus Engine) ⚙️

  * **Role:** Implements the core **Raft consensus logic**. It manages leader election, log replication, heartbeat mechanisms, and log safety checks.
  * **Key Component:** The **`RaftNode`** object, which handles the distributed log and cluster state (Leader/Follower/Candidate).

#### 2\. `distribute-java-cluster` (The Server Runtime) 🖥️

  * **Role:** Contains the main application entry points and deployment scripts. It initializes and runs the system.
  * **Key Components:** The **`RPCServer`** which handles both internal Raft communication (`RaftConsensusService`) and external client requests (`ExampleService`).

#### 3\. `ExampleStateMachine` (The Application Storage Layer) 💾

  * **Role:** This is the actual Key-Value store. It only accepts state-changing updates from the Raft Consensus Engine after they have been **committed** to the distributed log.
  * **Key Methods:** `applyData(byte[] dataBytes)` for committed writes, and methods for **Snapshotting** (`writeSnap`, `readSnap`) to manage log growth.

-----

### 3\. Quick Start (Local 3-Node Cluster)

You can quickly deploy a local 3-instance cluster for testing:

1.  **Deploy Cluster:**

    ```bash
    cd distribute-java-cluster
    sh deploy.sh
    ```

    This deploys three instances to the `distribute-java-cluster/env` directory, listening on ports **8051, 8052, and 8053**.

2.  **Test Write (SET):**

    ```bash
    cd env/client
    ./bin/run_client.sh "list://127.0.0.1:8051,127.0.0.1:8052,127.0.0.1:8053" mykey myvalue
    ```

3.  **Test Read (GET):**

    ```bash
    ./bin/run_client.sh "list://127.0.0.1:8051,127.0.0.0:8052,127.0.0.1:8053" mykey
    ```

-----

### 4\. Data Flow and Consistency

All state-changing operations are processed through the Raft Leader to guarantee strong consistency:

1.  **Client Request:** Sent to the cluster. If received by a Follower, it is **redirected** to the Leader.
2.  **Log Entry:** The Leader appends the operation to its Raft Log and initiates **replication** to all Followers.
3.  **Commit & Apply:** Once a **majority** of nodes have replicated the entry, it is marked **Committed**, and only then is it applied to the `ExampleStateMachine` on all nodes.
