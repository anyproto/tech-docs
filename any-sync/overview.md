# Protocol overview

Any-Sync is an open-source protocol that enables local first communication and collaboration based on CRDTs. There are two important differentiators of Any-Sync:
- It is designed around creators’ controlled keys
- It is focused on bringing high-performance communication and collaboration at scale

Any-Sync fulfills the seven ideals of [local first software](https://www.inkandswitch.com/local-first/):
- **No spinners**: your work at your fingertips. Any-Sync keeps the primary copy of each space on the local device. Data synchronization with other devices happens quietly in the background - allowing you to operate with your data at your fingertips.
- **Your work is not trapped on one device.** Users can easily work on different devices. Each device keeps data in local storage, synchronization between devices happens in the background using CRDTs to resolve conflicts.
- **The network is optional.** Everything works offline. Internet is not required to synchronize the data: Any-Sync allows users to do it via local network as well. Still, there is a role for the network - it works as additional backup, helps with peer discovery and especially solves the closed-laptop problem (you made changes on laptop, when your phone was offline, the changes can either sync when both devices are online or via backup node).
- **Seamless collaboration with your colleagues.** Achieving this goal is one of the biggest challenges in realizing local-first software, that’s why Any-Sync is built with CRDTs. So each device resolves conflicts independently.
- **The Long Now.** Because you have a local-first application, you can use it on your computer even if the software author disappears. This is also strengthened by open data standards and open code.
- **Security and privacy by default.** Any-Sync uses end-to-end encryption so that backup nodes store encrypted data that they cannot read. Conflict resolution happens on-device. The keys are controlled by users.
- **You retain ultimate ownership and control.** In the local first ideals this meant that you have local data, so you have ultimate ownership and control. To realize the idea of ultimate ownership we added creator controlled keys to Anytype.

Additional two ideals that Any-Sync adds:
- **Creators’ controlled keys.** Creators control encryption keys; there is no central registry of users (we don’t even ask your email). We added an option to self-host your backup to support full autonomy of users from the network.
- **Open Source.** Any-Sync protocol is open source, so all claims about how it works are independently verifiable.

We have released Anytype - the interface that is built on Any-Sync protocol. Users of Anytype can create spaces - graph-based databases with modular UI. Each space has unique access rights.

## Introduction

We designed Any-Sync out of a strong conviction that the Internet today is akin to a nervous system of humanity - today it is cloud based, so all the neurons can communicate only via servers that are controlled by different elites. We envision a “no-one in between” local first alternative would be a much better foundation for communication on the internet. For this we’ve built Any-Sync to support fast and scalable synchronization of discussions, communities and apps.

Features:

* Conflict-free data replication across multiple devices and agents
* Built-in end-to-end encryption
* Cryptographically verifiable history of changes
* Adoption to frequent operations (high performance)
* Reliable and scalable infrastructure
* Simultaneous support of p2p and remote communication

## Protocol explanation

### Data representation

![CRDT](../assets/crdt.gif)

#### Objects

`any-sync` is designed to synchronize digital objects that are structured as [Conflict-free Replicated Data](https://en.wikipedia.org/wiki/Conflict-free\_replicated\_data\_type) [Directed Acyclic Graphs](https://en.wikipedia.org/wiki/Directed\_acyclic\_graph) (DAGs). In this representation, each object is considered as a root, representing its initial state, and captures all the subsequent changes made to the object over time. Put simply, in `any-sync`, the objects serve as a comprehensive record of the complete history of the associated changes.

The changes, which don’t have changes after them, are object heads.

#### Conflict resolution

`any-sync` is specifically designed to support collaboration among multiple devices and agents, which can result in situations where objects have multiple "heads." In this context, a head refers to the local state of the object as observed by each device or agent involved.

When a user makes changes to an object that has multiple heads, the new change will incorporate and reference all of these heads. This process effectively "merges" several branches together, resulting in a unified and singular state for the object.

During the process of retrieving the current state of an object, the protocol begins with a specific head and follows Content IDs (CIDs) to construct the sequence of changes. If there are multiple heads or several possible ways to identify the previous change, a topological sort is employed. This sort relies on the hash values associated with the changes to determine the order in which the changes occurred, ultimately establishing the correct sequence of changes.

#### Snapshots

To enhance the efficiency of retrieving the current state, the protocol employed by `any-sync` uses a probability-based mechanism that can transform a change into a snapshot. When a snapshot is created, there is no longer a need to analyze the changes that occurred before it in order to reconstruct the current state of the object. This optimization allows for faster retrieval by skipping the analysis of preceding changes when a snapshot is available.

`any-sync` is a protocol that is independent of specific client implementations, making it client-agnostic. It offers a mechanism to traverse objects, allowing applications to build their own application state based on the objects received through the protocol. In other words, applications using `any-sync` can utilize the objects received from it to construct their own internal state, tailored to their specific requirements and functionalities.

#### Files

`any-sync`'s file processing is a two-part solution: data storage and data retrieval.

It uses [IPLD](https://ipld.io/) data structures to store files' data. Files are stored locally on users' devices or on file-nodes on the external network, which means the protocol works with a simple network topology.

`any-sync` has its own approach for file retrieval, as opposed to [bitswap](https://docs.ipfs.tech/concepts/bitswap/), which is designed for networks with complex topologies.

#### Spaces

![Space](../assets/space.png)

A Space is a collection of digital objects with an Access Control List (ACL), which allows the user to define who can read and modify its contents.

In the simplest scenario, each space holds the data of a single user, and this space can be accessed from multiple devices belonging to that user. However, in situations in which multiple agents are collaborating in the same space, the space stores data from and on the devices of each user who has access to it. In these cases, the permissions for accessing and managing the data within the spaces are determined by ACLs.

Spaces are stored locally on user’s devices or on sync nodes on the external network.

In case of ACL changes, a sync node requests a validation from a consensus node. If the ACL change is valid (linked list without conflicts), the sync node updates the space ACL and sends the change to other sync nodes; otherwise, the sync node rejects the change. Hence, updating ACL requires a connection to the external network. If there is no connection, the change will be rejected.

#### Encryption

Each user has private and public keys which are used for signing, encrypting, and decrypting.

`any-sync` uses the [Ed25519](https://en.wikipedia.org/wiki/EdDSA) algorithm for signing to provide strong cryptographic security, while still keeping the performance overhead low.

Every time a user modifies the data, the changes are both encrypted and signed using their own private key. When these changes are synchronized with other devices or sync-nodes, they are verified using a shared public key. However, only the user who possesses the private key can access the content of these changes.

### Infrastructure

![Infrastructure](../assets/infrastructure.png)

While `any-sync` works locally on user’s devices and in local p2p networks, an Infrastructure layer is needed to provide external data storage and backups, as well as seamless collaboration between agents in different networks.

`any-sync` protocol works the same way on the user’s devices and on the infrastructure side. So every node could be seen as a peer.

#### Nodes

The infrastructure side consists of four types of nodes: sync, file, consensus and coordinator nodes.

* _Sync nodes_ store and process spaces. Each space is served by several sync nodes in order to increase the reliability of infrastructure.
* _File nodes_ store and process files based on IPLD data structure.
* _Consensus nodes_ monitor ACL changes and validate them.
* _Coordinator nodes_ store and update the infrastructure configuration: they manage the list of nodes, provide clients with addresses of nodes for spaces, provide nodes with addresses of replicas, etc.

#### Load distribution

[Load distribution](https://github.com/anyproto/go-chash) is implemented with a combination of modular hashing and consistent hashing. The algorithm distributes partitions (groups of spaces) between nodes using consistent hashing, then uses modular hashing to determine the partition for space (and thus the node for space).

The algorithm limits the load on a node, which makes distribution more or less even.

Adding a new node leads to a change in infrastructure configuration. This event starts the resharding process, when spaces and data within them are transferred between nodes according to the new configuration.

### Peer retrieval

![Hybrid peer retrieval](../assets/peer_retrieval.png)

#### Local p2p

To find peers in the local network and communicate with them without connecting to external infrastructure, any-sync relies on [mDNS protocol](https://en.wikipedia.org/wiki/Multicast\_DNS).

User’s device sends an mDNS query asking for the IP address of devices with a specific hostname. Other devices on the network that have that hostname will respond with their IP address.

Once a client has the IP address of a peer device, any-sync tries to retrieve the spaces that exist on this device.

If the current user has access to spaces on a peer device (based on space ACL), then regular message interchange starts.

Files stored on a peer device can be retrieved directly from them as well.

This allows for efficient and direct communication between devices on the same network without the need for a remote server or internet connection.

#### Global networks

`any-sync` on the user's device stores a configuration with the address of external infrastructure. It requests a coordinator node for the address of the sync node, which is assigned to serve the user's space.

Once a client receives the address of the node for a space, it subscribes for updates and sends changes to it.

Files stored on the external infrastructure are retrieved from file nodes.

Moreover, a hybrid mode is possible if a user has two devices with any-sync in the same local network. Let’s say the first device has access to external infrastructure, while the second device does not.

In this case, the first device acts as a bridge. It exchanges data with the local peer and translates changes made on the second device to the external storage.

It also works the other way: when the first device receives updates from the remote sync-node, it sends them to the second device via local p2p connection.

## Development background

The importance of developing and maintaining a completely new protocol was clear to us, and we made the decision after carefully considering existing solutions.

We evaluated the implementation of algorithms like RGA, Logoot, LSEQ, Automerge, and others. Although they share similar ideas regarding data representation, such as a logical ordering of object operations, they don't offer content identifiers and encryption schemes.

Additionally, they are designed to work with plaintext data, while any-sync relies on a binary format to support complex data structures.

We thus designed data structures that fit our requirements. Notably, none of the solutions mentioned support hybrid networking, which would require us to build something on top of them.

We also considered using IPFS, but its approach to content identifiers didn't meet our use cases. While IPFS could address our requirements for hybrid networking, it uses CID to describe documents, and this CID changes with each modification. However, we need documents to have stable identifiers for linking and other purposes.

Creating stable identifiers and implementing logic to merge different document states on top of IPFS would be too complex, so we chose a straightforward approach.

## Next steps

We are excited to announce the early development stage release of `any-sync` to the community.

In the upcoming phases of development, we have planned to implement the following key features:

* Full Access Control List (ACL) support for multiple agents: This enhancement will enable intricate collaboration scenarios by providing comprehensive control over access permissions and privileges among different agents involved in the synchronization process.
* Multiple service providers: We intend to introduce support for multiple service providers, facilitating seamless communication between diverse remote infrastructures. This feature will enhance interoperability and enable users to connect and collaborate across various infrastructure providers.
* Global peer-to-peer (p2p) communication: `any-sync` will be enhanced to enable direct communication and connection between devices without the need for a remote infrastructure. This advancement will empower users with the ability to establish direct p2p connections, fostering faster and more efficient data synchronization.

We are committed to continually improving `any-sync` and are thrilled to bring these exciting features to fruition in the near future, further enhancing its capabilities for the benefit of the community.
