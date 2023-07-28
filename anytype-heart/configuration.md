# Configuration

You can generate basic configuration for your nodes and network with [`any-sync-network`](https://github.com/anyproto/any-sync-tools) tool.

## Network configuration options

```yaml
# required; describes any-sync network topology
networkId: # required; network public key
nodes:     # required; an array of nodes
  - peerId:    # required; node's account peer ID
    addresses: # required; an array of node's addresses
    types:     # required; an array of node's types,
               # allowed values: tree (for sync nodes), file, coordinator
```
