# Self-hosting

We believe that any software should support fundamental digital freedoms. With the rise of cryptography and computer systems, it is now possible to guarantee these freedoms in the world of bits: privacy of thoughts, freedom of speech, right to authorship, and autonomy from software providers. These rights can be encoded into the code, which when open, can be freely verified by anyone. This way, trust among users and developers can be established.

This is our way. By opening our source code, we ensure that our users have complete autonomy and independence from the Any Association. They retain the ability to analyze, compile, and run each software component on their personal machines without relying on external parties. This guarantees uninterrupted access to the tools and data they generate and store, shielding them from any potential restrictions.

This article will help you to self-host Any-Sync on your own infrastructure for personal use.

## Prerequisites

> There is a [docker-compose](https://github.com/anyproto/any-sync-dockercompose). This image is suitable for running your personal self-hosted any-sync network for home usage. If you plan to self-host a heavily used any-sync network, please consider other options.
>
> Also, there are [ansible repository](https://github.com/anyproto/ansible-anysync) and [puppet module](https://forge.puppetlabs.com/modules/anyproto/anysync/readme).
>
> If you want to proceed with a manual setup, follow the steps below.

To ensure compatibility, please use Go version `1.22` for building `any-sync-*` and `anytype-heart`.

You will need a MongoDB to run Any-Sync Coordinator and Consensus Nodes, and an S3-compatible object storage and Redis to run Any-Sync File Node.

Some of the possible S3-compatible solutions are:
- [MinIO](https://min.io/docs/minio/linux/operations/install-deploy-manage/deploy-minio-single-node-single-drive.html)
- [Seaweedfs](https://seaweedfs.github.io/)

> Please note that MongoDB should run in replica set mode, even if you are running only one instance. You can find more information about MongoDB replica sets [here](https://docs.mongodb.com/manual/replication/).
>
> Also, make sure that your Redis instance has [Bloom Filter](https://redis.io/docs/latest/develop/data-types/probabilistic/bloom-filter/) module included. As an option, you can use [redis-stack](https://github.com/redis-stack/redis-stack) distribution which has this and other modules built in.


## Steps

1. Decide what network topology you will need for `any-sync`. A set of coordinator, sync and file nodes, one of each, will be enough for personal use.
2. Set up MongoDB, S3-compatible object storage and Redis and save their properties to display them in configuration. You will need:
   * MongoDB:
     * Connection URL
     * Database name
   * S3 Bucket:
     * Endpoint (in case you proceed with S3-compatible object storage)
     * Region name
     * Profile
     * Bucket name
     * Credentials (see [`aws-sdk-go` documentation](https://pkg.go.dev/github.com/aws/aws-sdk-go#readme-configuring-credentials))
   * Redis:
     * Connection URL
3.  Create configurations for future `any-sync-*` nodes and `anytype-heart`.\
    You can generate basic configuration for your nodes with [`any-sync-network`](https://github.com/anyproto/any-sync-tools/tree/main/any-sync-network) tool, or you can create more complex configurations on your own (please see [Any-Sync Configuration](../any-sync/configuration.md "mention") for details).

    If you decide to use `any-sync-network` tool, please run following commands and use interactive CLI to generate configuration files:

    ```bash
    go install github.com/anyproto/any-sync-tools/any-sync-network@latest
    any-sync-network create
    ```

4. Download and build [`any-sync-coordinator`](https://github.com/anyproto/any-sync-coordinator), [`any-sync-node`](https://github.com/anyproto/any-sync-node), [`any-sync-filenode`](https://github.com/anyproto/any-sync-filenode), and [`any-sync-consensusnode`](https://github.com/anyproto/any-sync-consensusnode); see `README.md` inside each repo for details.
5. Run nodes using proper configuration files.

    If you generated them with `any-sync-network` tool, use:
   * `coordinator.yml` for `any-sync-coordinator` node,
   * `consensus.yml` for `any-sync-consensusnode`,
   * `sync_N.yml` for each of `any-sync-node`s,

      (🚨 create a `db` folder in the current working directory of `any-sync-node`, as it is required by the generated configuration)
   * `file_N.yml` for each of `any-sync-filenode`s.
6. Now you can configure your client apps to use your network configuration. If you generated it with `any-sync-network` tool, use `heart.yml`.

## Adding consensus node

> This information is important for those who started self-hosting without consensus node. If you are starting from scratch, please refer to the [Steps](#steps) section.

A new type of node has appeared in the `any-sync` network: `any-sync-consensusnode`. It is essential for the proper functioning of the network. If you are updating your network, please ensure that you add this node to your configuration. Please make sure of the following:

1. You have a running MongoDB replica set. It is possible to run a single MongoDB instance in replica set mode.
2. You have created a configuration for `any-sync-consensusnode` and added it to your network configuration. You can use the `any-sync-network` tool to generate it.
3. You have updated your network configuration to include `any-sync-consensusnode`. You can update the configuration files on each node, or you can use `any-sync-confapply` from `any-sync-coordinator/bin`:

    ```
    any-sync-confapply -n network.yml
    ```


## Conditions

`any-sync` networks are permissionless. This means that anyone who knows the IP and port of the coordinator node can connect to it, create new spaces, or download existing encrypted objects. Data inside spaces is always secure, as it requires space encryption keys to read it. Nodes do not store and do not have access to these encryption keys and cannot decrypt the data. Refer to the [Any-Sync Protocol overview](../any-sync/overview.md "mention") for additional details.

Middleware library `anytype-heart` and Anytype clients `anytype-ts`, `anytype-swift` and `anytype-kotlin` are licensed under [Any Source Available License 1.0](https://networks.any.coop).

It allows Non-Commercial use, but if you plan a Commercial use for your self-hosted instance, you will need to obtain a separate use grant from us. Please contact [license@any.coop](mailto:license@any.coop) in this case first.
