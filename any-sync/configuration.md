# Configuration

You can generate basic configuration for your nodes with [`any-sync-network`](https://github.com/anyproto/any-sync-tools/tree/main/any-sync-network) tool.

## Common nodes configuration options

These options are common for all types of Any-Sync nodes.

```yaml
# required; Account, which is used by the node to communucate inside the network 
account: 
  peerId:          # required; public key
  peerKey:         # required; private key
  signingKey:      # required; private key, 
                   # coordinator node uses private key of the network
                   # sync and file nodes use their peerKey

# required; tcp/yamux network configuration
yamux:
  listenAddrs:     # required; an array of address which node should listen
  writeTimeoutSec: # required
  dialTimeoutSec:  # required

# required; udp/quic network configuration
quic:
  listenAddrs:     # required; an array of address which node should listen
  writeTimeoutSec: # required
  dialTimeoutSec:  # required

# required; describes any-sync network topology
network:
  id:              # required; configuration ID
  networkId:       # required; network public key
  nodes:           # required; an array of nodes
    - peerId:        # required; node's account peer ID
      addresses:     # required; an array of node's addresses
      types:         # required; an array of node's types,
                     # allowed values: tree (for sync nodes), file, consensus, coordinator
  creationTime:    # required; configuration's creation time

# required; a path where current configuration is stored
networkStorePath: 

# optional; dRPC parameters
drpc:
  stream:
    maxMsgSizeMb:  # required

# optional; Prometheus monitoring listener configuration
metric:
  addr:            # optional; Prometheus monitoring listener address
```

## Coordinator Node configuration options

These options are specific for Any-Sync Coordinator nodes.

```yaml
# required; MongoDB configuration
mongo:
  connect:            # required; MongoDB connection URL
  database:           # required; MongoDB database name

# required; Space status update configuration
spaceStatus:
  runSeconds:         # required; Space status update frequency
  deletionPeriodDays: # required; Space data deletion frequency
  ```

## Sync Node configuration options

These options are specific for Any-Sync Sync nodes.

```yaml
# required; Network configuration update interval
# Descibes how often the node requests network configuration from the coordinator node
networkUpdateIntervalSec: 

# required; Space cache settings
space:
  gcTTL:             # required; Space cache time-to-live in memory
  syncPeriod:        # required

# required; Space data storage configuration
storage:
  path:              # required; Local path for space data storage

# required; Direct inter-node (without coordinator) sync configuration
nodeSync:
  hotSync:           # required
    simultaneousRequests: # required
  syncOnStart:       # required; flag (boolean)
  periodicSyncHours: # required

# required; Log configuration
log:
  production:        # required; flag (boolean)
  defaultLevel:      # required
  namedLevels:       # required

# optional; Debug API listener configuration, must not be public
apiServer:
  listenAddr:        # optional; Debug API listener address
```

## File Node configuration options

These options are specific for Any-Sync File nodes.

```yaml
# required; S3 configuration
s3Store:
  endpoint:  # optional; endpoint for S3-compatible object storage
  region:     # required; S3 region name
  profile:    # required; S3 profile name
  bucket:     # required; S3 bucket name
  maxThreads: # required; maximum threads number

# required; Redis configuration
redis:
  isCluster:  # required; Redis cluster flag (boolean)
  url:        # required; Redis connection URL

# required; Network configuration update interval
# Descibes how often the node requests network configuration from the coordinator node
networkUpdateIntervalSec:

# optional; File node dev-mode storage path
# File Node can be built in dev mode (`make build-dev`), bypasses S3, not reliable
fileDevStore:
  path:       # optional; Local path for files storage
```
## Consensus Node configuration options

These options are specific for Any-Sync Consensus nodes.

```yaml
# required; MongoDB configuration
mongo:
  connect:       # required; MongoDB connection URL
  database:      # required; MongoDB database name
  logCollection: # required; MongoDB collection name for logs

# required; Network configuration update interval
# Descibes how often the node requests network configuration from the coordinator node
networkUpdateIntervalSec: 
```