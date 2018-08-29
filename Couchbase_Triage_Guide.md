# Overview
DC/OS Couchbase is an automated service that makes it easy to deploy and manage Couchbase on DC/OS.

## General Couchbase Triage Steps

1. Collect a DC/OS log bundle
2. Collect Scheduler logs - stderr & stdout.
3. Collect Node Logs - stderr & stdout.
Pro Tip: ​dcos task log --all couchbase-dev > couchbase-dev.log

### Couchbase Specific Logs
All logs from /var/lib/couchbase/logs

## General framework troubleshooting
https://docs.mesosphere.com/services/ops-guide/troubleshooting/

## Specific Scenarios:

### Scenario: node task becomes unresponsive

There are situations where a complete node replacement can become necessary, e.g the node task becomes unresponsive. We will use the `couchbase console` and the `dcos couchbase` cli to get things back to normal.

We first need to `failover` to the remaining couchbase nodes.

![Resources](img/replace02.png)

Confirm the failover request.

![Resources](img/replace04.png)

Once the failover is complete a `rebalance` is required.

![Resources](img/replace05.png)

With that our ill node is removed from the couchbase cluster.

![Resources](img/replace06.png)

Next we use the `dcos cli` to replace the node by using the following command.

```
dcos couchbase pod replace data-1
```

Looking at the mesos console we see that data-1-node got killed and a new one created.

![Resources](img/replace03.png)

Back in the coucbase console we need to `add` the newly created node to the couchbase cluster. Since in our sample we replace a data node therefore select the `data checkbox` only.

![Resources](img/replace07.png)

The node is added to the cluster and a another `rebalance` is required.

![Resources](img/replace08.png)

After that we are back to normal.

![Resources](img/replace09.png)


### Scenario: Backup and Restore

Sometimes it will be prudent to ask customer to backup their data (which can be restored later)

For backup and restore we leverage the `cbbackupmgr` tool that comes with Couchbase enterprise.

A dedicate `backpumgr service` node has to be launched, see respective section in the dcos couchbase service configuration. It provides the volume to store the incremental snapshots of the database, and provides tasks for the various `cbbackupmgr` commands (backup, restore, list, merge)

The backupmgr node is setup with a connection to an s3 compatible store (default is minio). The tasks use `aws s3 sync` command to keep the incremental snapshots on the backupmgr node and in the connected s3 bucket in sync.

**Note:** In your s3 compatible store you will have to create a bucket with the name that you specified in the backupmgr configuration.

### dcos couchbase plan start backupmgr-backup
Creates a incremental snapshot and syncs it with s3 bucket.

### dcos couchbase plan start backupmgr-restore
Syncs with s3 bucket then does the restore. Empty couchbase buckets have to be created before the restore.

### dcos couchbase plan start backupmgr-list
Lists the snapshots. You find it in the sdtout of the task.

### dcos couchbase plan start backupmgr-merge
The merge command allows you to merge snapshots together. Snapshots have a timestamp, you can get them via the list command.

```
dcos couchbase plan start backupmgr-merge -p MERGE_START=<start-time-stamp> -p MERGE_END=<end-time-stamp>
```

After the merge is completed it is also synced with the s3 bucket. After that both only contain the merged snapshot.

### Scenario: Identify Deployment Topology

Couchbase framework supports 2 deployment topologies:

1. Data nodes have all the Couchbase server personalities (for development purposes only)

![Resources](img/dev_deploy.png)

2. Each couchbase server personality runs in its own container (Production deployment)

![Resources](img/prod_deploy.png)

*To verify*:
- Get Yml from the scheduler logs

