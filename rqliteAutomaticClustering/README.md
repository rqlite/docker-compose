# Docker Compose


## Automatic clustering with 3 nodes

This guide demonstrates how to set up an automatically configured three-node rqlite cluster using Docker Compose.
The automatic clustering feature simplifies the initial configuration, allowing new nodes to discover and join the existing cluster with minimal manual intervention.
The setup explicitly instructs each joining node to connect to existing nodes, ensuring a robust and self-forming cluster.


### Step-by-Step Setup


#### 1. Project directory `rqliteAutomaticClustering`

The entire setup is contained within a dedicated `rqliteAutomaticClustering` folder, which will house the `compose.yaml` file.


#### 2. Create the data persistence folder

A `rqlite-data` directory is created to ensure data persistence for each rqlite node.
The `compose.yaml` file configures separate subfolders within `rqlite-data` for each node (e.g., `./rqlite-data/myrqlite-node-1`), which is crucial for preventing data conflicts and maintaining individual node states.

```bash
$ mkdir -p rqlite-data
```


#### 3. Review the `rqliteAutomaticClustering` `compose.yaml` file

This file defines our three rqlite services, `myrqlite-service-1`, `myrqlite-service-2`, and `myrqlite-service-3`.
Notice how `myrqlite-service-1`, `myrqlite-service-2` and `myrqlite-service-3` use the `-join` command to connect together.
Command `-bootstrap-expect` and `-join`: This is the core of the automatic clustering.
`-bootstrap-expect 3` flag tells each node to wait until 3 nodes are present before attempting to form a cluster.
This helps prevent split-brain scenarios in a multi-node setup.
`-join myrqlite-service-1:4002,myrqlite-service-2:4002,myrqlite-service-3:4002` instructs each node to attempt to join the cluster via the specified raft addresses of the other nodes.
Docker Compose's internal DNS resolution allows nodes to refer to each other by their service names.
Take a look at its contents:

```bash
$ cat compose.yaml
```

```yaml
# Created: 2025-06-10 13:31:10
# Updated:
# Language: Docker Compose version v2.36.2
# Images:
#   - rqlite/rqlite:8
# Project: rqlite Automatic Clustering

name: rqliteAutomaticClustering

services:

  myrqlite-service-1:
    image: rqlite/rqlite:latest
    container_name: myrqlite-container-1
    hostname: myrqlite-host-1
    volumes:
      - ./rqlite-data/myrqlite-node-1:/rqlite/file
    ports:
      - "4001:4001"
      - "4002:4002"
    command: -bootstrap-expect 3 -join myrqlite-service-1:4002,myrqlite-service-2:4002,myrqlite-service-3:4002
    environment:
      NODE_ID: myrqlite-node-1
      HTTP_ADDR: myrqlite-host-1:4001
      RAFT_ADDR: myrqlite-host-1:4002
      SQLITE_EXTENSIONS: "sqlean,sqlite-vec,misc"

  myrqlite-service-2:
    image: rqlite/rqlite:latest
    container_name: myrqlite-container-2
    hostname: myrqlite-host-2
    volumes:
      - ./rqlite-data/myrqlite-node-2:/rqlite/file
    ports:
      - "4011:4001"
      - "4012:4002"
    command: -bootstrap-expect 3 -join myrqlite-service-1:4002,myrqlite-service-2:4002,myrqlite-service-3:4002
    environment:
      NODE_ID: myrqlite-node-2
      HTTP_ADDR: myrqlite-host-2:4001
      RAFT_ADDR: myrqlite-host-2:4002
      SQLITE_EXTENSIONS: "sqlean,sqlite-vec,misc"

  myrqlite-service-3:
    image: rqlite/rqlite:latest
    container_name: myrqlite-container-3
    hostname: myrqlite-host-3
    volumes:
      - ./rqlite-data/myrqlite-node-3:/rqlite/file
    ports:
      - "4021:4001"
      - "4022:4002"
    command: -bootstrap-expect 3 -join myrqlite-service-1:4002,myrqlite-service-2:4002,myrqlite-service-3:4002
    environment:
      NODE_ID: myrqlite-node-3
      HTTP_ADDR: myrqlite-host-3:4001
      RAFT_ADDR: myrqlite-host-3:4002
      SQLITE_EXTENSIONS: "sqlean,sqlite-vec,misc"
```


#### 4. Launch the rqlite service

Using docker `compose up -d` brings up all three rqlite services in detached mode, allowing them to run in the background.
The output clearly shows the image pulling and container startup process.

```bash
$ docker compose up -d
```

```text
[+] Running 10/10
 ✔ myrqlite-service-3 Pulled                                                          7.5s
 ✔ myrqlite-service-1 Pulled                                                          7.5s
   ✔ fe07684b16b8 Already exists                                                      0.0s
   ✔ 0ab638900fe9 Pull complete                                                       2.2s
   ✔ ee0ca339bb46 Pull complete                                                       2.2s
   ✔ bbb0f06e65bb Pull complete                                                       4.0s
   ✔ af50c63557f2 Pull complete                                                       4.1s
   ✔ 36f51aa6517e Pull complete                                                       4.1s
   ✔ be1e3eb83208 Pull complete                                                       4.2s
 ✔ myrqlite-service-2 Pulled                                                          9.2s
[+] Running 4/4
 ✔ Network rqliteautomaticclustering_default  Created                                 0.0s
 ✔ Container myrqlite-container-3             Started                                 0.5s
 ✔ Container myrqlite-container-1             Started                                 0.5s
 ✔ Container myrqlite-container-2             Started                                 0.4s
```


#### 5. Verify the service status

`docker compose ps` command is essential for confirming that all containers are running as expected, along with their exposed ports.

```bash
$ docker compose ps
```

```text
NAME                   IMAGE                  COMMAND                  SERVICE              CREATED         STATUS         PORTS
myrqlite-container-1   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-1   2 minutes ago   Up 2 minutes   0.0.0.0:4001-4002->4001-4002/tcp, [::]:4001-4002->4001-4002/tcp
myrqlite-container-2   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-2   2 minutes ago   Up 2 minutes   0.0.0.0:4011->4001/tcp, [::]:4011->4001/tcp, 0.0.0.0:4012->4002/tcp, [::]:4012->4002/tcp
myrqlite-container-3   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-3   2 minutes ago   Up 2 minutes   0.0.0.0:4021->4001/tcp, [::]:4021->4001/tcp, 0.0.0.0:4022->4002/tcp, [::]:4022->4002/tcp
```


#### 6. Inspect the cluster logs

The combined logs provide valuable insight into the cluster formation process.
You can observe messages related to node starting, joining attempts, successful bootstrapping, and leader election.
The logs clearly show myrqlite-node-3 becoming the leader in this particular run.

```bash
$ docker compose logs
```

```text
myrqlite-container-1  |
myrqlite-container-1  |             _ _ _
myrqlite-container-1  |            | (_) |
myrqlite-container-1  |   _ __ __ _| |_| |_ ___
myrqlite-container-1  |  | '__/ _  | | | __/ _ \   The lightweight, distributed
myrqlite-container-1  |  | | | (_| | | | ||  __/   relational database.
myrqlite-container-1  |  |_|  \__, |_|_|\__\___|
myrqlite-container-1  |          | |               www.rqlite.io
myrqlite-container-1  |          |_|
myrqlite-container-1  |
myrqlite-container-2  | [rqlited] 2025/06/16 18:55:54 rqlited starting, version v8.37.4, SQLite 3.49.1, commit 4870c35c694f3173b83c45a275d7a943c8187535, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-2  | [rqlited] 2025/06/16 18:55:54 go1.24.4, target architecture is amd64, operating system target is linux
myrqlite-container-2  | [rqlited] 2025/06/16 18:55:54 launch command: /bin/rqlited -node-id myrqlite-node-2 -http-addr myrqlite-host-2:4001 -http-adv-addr myrqlite-host-2:4001 -raft-addr myrqlite-host-2:4002 -raft-adv-addr myrqlite-host-2:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip -bootstrap-expect 3 -join myrqlite-service-1:4002,myrqlite-service-2:4002,myrqlite-service-3:4002 /rqlite/file/data
myrqlite-container-3  |
myrqlite-container-3  |             _ _ _
myrqlite-container-3  | [rqlited] 2025/06/16 18:55:54 rqlited starting, version v8.37.4, SQLite 3.49.1, commit 4870c35c694f3173b83c45a275d7a943c8187535, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-3  | [rqlited] 2025/06/16 18:55:54 go1.24.4, target architecture is amd64, operating system target is linux
myrqlite-container-3  |            | (_) |
myrqlite-container-3  |   _ __ __ _| |_| |_ ___
myrqlite-container-3  |  | '__/ _  | | | __/ _ \   The lightweight, distributed
myrqlite-container-3  |  | | | (_| | | | ||  __/   relational database.
myrqlite-container-3  |  |_|  \__, |_|_|\__\___|
myrqlite-container-3  |          | |               www.rqlite.io
myrqlite-container-3  |          |_|
myrqlite-container-1  | [rqlited] 2025/06/16 18:55:54 rqlited starting, version v8.37.4, SQLite 3.49.1, commit 4870c35c694f3173b83c45a275d7a943c8187535, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-3  |
myrqlite-container-1  | [rqlited] 2025/06/16 18:55:54 go1.24.4, target architecture is amd64, operating system target is linux
myrqlite-container-3  | [rqlited] 2025/06/16 18:55:54 launch command: /bin/rqlited -node-id myrqlite-node-3 -http-addr myrqlite-host-3:4001 -http-adv-addr myrqlite-host-3:4001 -raft-addr myrqlite-host-3:4002 -raft-adv-addr myrqlite-host-3:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip -bootstrap-expect 3 -join myrqlite-service-1:4002,myrqlite-service-2:4002,myrqlite-service-3:4002 /rqlite/file/data
myrqlite-container-3  | [mux] 2025/06/16 18:55:54 mux serving on 172.18.0.4:4002, advertising myrqlite-host-3:4002
myrqlite-container-3  | [rqlited] 2025/06/16 18:55:54 no preexisting node state detected in /rqlite/file/data, node may be bootstrapping
myrqlite-container-3  | [cluster] 2025/06/16 18:55:54 service listening on myrqlite-host-3:4002
myrqlite-container-3  | [http] 2025/06/16 18:55:54 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-1  | [rqlited] 2025/06/16 18:55:54 launch command: /bin/rqlited -node-id myrqlite-node-1 -http-addr myrqlite-host-1:4001 -http-adv-addr myrqlite-host-1:4001 -raft-addr myrqlite-host-1:4002 -raft-adv-addr myrqlite-host-1:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip -bootstrap-expect 3 -join myrqlite-service-1:4002,myrqlite-service-2:4002,myrqlite-service-3:4002 /rqlite/file/data
myrqlite-container-1  | [mux] 2025/06/16 18:55:54 mux serving on 172.18.0.3:4002, advertising myrqlite-host-1:4002
myrqlite-container-1  | [rqlited] 2025/06/16 18:55:54 no preexisting node state detected in /rqlite/file/data, node may be bootstrapping
myrqlite-container-1  | [cluster] 2025/06/16 18:55:54 service listening on myrqlite-host-1:4002
myrqlite-container-1  | [http] 2025/06/16 18:55:54 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-1  | [http] 2025/06/16 18:55:54 service listening on 172.18.0.3:4001
myrqlite-container-1  | [store] 2025/06/16 18:55:54 opening store with node ID myrqlite-node-1, listening on myrqlite-host-1:4002
myrqlite-container-1  | [store] 2025/06/16 18:55:54 ensuring data directory exists at /rqlite/file/data
myrqlite-container-1  | [store] 2025/06/16 18:55:54 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-1  | [snapshot-store] 2025/06/16 18:55:54 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-3  | [http] 2025/06/16 18:55:54 service listening on 172.18.0.4:4001
myrqlite-container-3  | [store] 2025/06/16 18:55:54 opening store with node ID myrqlite-node-3, listening on myrqlite-host-3:4002
myrqlite-container-3  | [store] 2025/06/16 18:55:54 ensuring data directory exists at /rqlite/file/data
myrqlite-container-3  | [store] 2025/06/16 18:55:54 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-3  | [snapshot-store] 2025/06/16 18:55:54 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-3  | [store] 2025/06/16 18:55:54 0 preexisting snapshots present
myrqlite-container-3  | [store] 2025/06/16 18:55:54 raft log is 0 bytes at open, indexes are: first 0, last 0
myrqlite-container-3  | [db] 2025/06/16 18:55:54 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-3  | [rqlited] 2025/06/16 18:55:54 checking that supplied join addresses don't serve HTTP(S)
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-1:4002: no leader (did you forget to set -join-as?)
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-2:4002: no leader (did you forget to set -join-as?)
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-3:4002: no leader (did you forget to set -join-as?)
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:54 failed to join cluster at [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002], after 1 attempt(s)
myrqlite-container-1  | [store] 2025/06/16 18:55:54 0 preexisting snapshots present
myrqlite-container-1  | [store] 2025/06/16 18:55:54 raft log is 0 bytes at open, indexes are: first 0, last 0
myrqlite-container-1  | [db] 2025/06/16 18:55:54 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-1  | [rqlited] 2025/06/16 18:55:54 checking that supplied join addresses don't serve HTTP(S)
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-1:4002: no leader (did you forget to set -join-as?)
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-2:4002: no leader (did you forget to set -join-as?)
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-3:4002: store not open (did you forget to set -join-as?)
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:54 failed to join cluster at [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002], after 1 attempt(s)
myrqlite-container-3  | [cluster-bootstrap] 2025/06/16 18:55:54 succeeded notifying all targets: [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002]
myrqlite-container-2  |
myrqlite-container-3  | [store] 2025/06/16 18:55:56 reached expected bootstrap count of 3, starting cluster bootstrap
myrqlite-container-2  |             _ _ _
myrqlite-container-3  | [store] 2025/06/16 18:55:56 cluster bootstrap successful, servers: [{Voter myrqlite-node-1 myrqlite-host-1:4002} {Voter myrqlite-node-2 myrqlite-host-2:4002} {Voter myrqlite-node-3 myrqlite-host-3:4002}]
myrqlite-container-2  |            | (_) |
myrqlite-container-2  |   _ __ __ _| |_| |_ ___
myrqlite-container-2  |  | '__/ _  | | | __/ _ \   The lightweight, distributed
myrqlite-container-2  |  | | | (_| | | | ||  __/   relational database.
myrqlite-container-2  |  |_|  \__, |_|_|\__\___|
myrqlite-container-2  |          | |               www.rqlite.io
myrqlite-container-1  | [cluster-bootstrap] 2025/06/16 18:55:54 failed to notify all targets: [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002] (failed to notify node at myrqlite-service-3:4002: store not open, will retry)
myrqlite-container-2  |          |_|
myrqlite-container-1  | [raft] 2025/06/16 18:55:56 [WARN]  no known peers, aborting election
myrqlite-container-2  |
myrqlite-container-2  | [mux] 2025/06/16 18:55:54 mux serving on 172.18.0.2:4002, advertising myrqlite-host-2:4002
myrqlite-container-2  | [rqlited] 2025/06/16 18:55:54 no preexisting node state detected in /rqlite/file/data, node may be bootstrapping
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-1:4002: no leader (did you forget to set -join-as?)
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-2:4002: no leader (did you forget to set -join-as?)
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-3:4002: no leader (did you forget to set -join-as?)
myrqlite-container-3  | [cluster-join] 2025/06/16 18:55:56 failed to join cluster at [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002], after 1 attempt(s)
myrqlite-container-3  | [cluster-bootstrap] 2025/06/16 18:55:56 succeeded notifying all targets: [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002]
myrqlite-container-3  | [raft] 2025/06/16 18:55:56 [WARN]  heartbeat timeout reached, starting election: last-leader-addr= last-leader-id=
myrqlite-container-3  | [store] 2025/06/16 18:55:56 this node (ID=myrqlite-node-3) is now Leader
myrqlite-container-3  | [cluster-bootstrap] 2025/06/16 18:55:58 boot operation marked done
myrqlite-container-2  | [cluster] 2025/06/16 18:55:54 service listening on myrqlite-host-2:4002
myrqlite-container-3  | [rqlited] 2025/06/16 18:55:58 node HTTP API available at http://myrqlite-host-3:4001
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-1:4002: no leader (did you forget to set -join-as?)
myrqlite-container-3  | [rqlited] 2025/06/16 18:55:58 connect using the command-line tool via 'rqlite -H myrqlite-host-3 -p 4001'
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-2:4002: no leader (did you forget to set -join-as?)
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-3:4002: no leader (did you forget to set -join-as?)
myrqlite-container-1  | [cluster-join] 2025/06/16 18:55:56 failed to join cluster at [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002], after 1 attempt(s)
myrqlite-container-1  | [cluster-bootstrap] 2025/06/16 18:55:56 succeeded notifying all targets: [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002]
myrqlite-container-1  | [store] 2025/06/16 18:55:56 reached expected bootstrap count of 3, starting cluster bootstrap
myrqlite-container-1  | [store] 2025/06/16 18:55:56 cluster bootstrap successful, servers: [{Voter myrqlite-node-1 myrqlite-host-1:4002} {Voter myrqlite-node-3 myrqlite-host-3:4002} {Voter myrqlite-node-2 myrqlite-host-2:4002}]
myrqlite-container-1  | [store] 2025/06/16 18:55:56 node myrqlite-node-3 is now Leader
myrqlite-container-1  | [cluster-bootstrap] 2025/06/16 18:55:58 boot operation marked done
myrqlite-container-1  | [rqlited] 2025/06/16 18:55:58 node HTTP API available at http://myrqlite-host-1:4001
myrqlite-container-1  | [rqlited] 2025/06/16 18:55:58 connect using the command-line tool via 'rqlite -H myrqlite-host-1 -p 4001'
myrqlite-container-2  | [http] 2025/06/16 18:55:54 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-2  | [http] 2025/06/16 18:55:54 service listening on 172.18.0.2:4001
myrqlite-container-2  | [store] 2025/06/16 18:55:54 opening store with node ID myrqlite-node-2, listening on myrqlite-host-2:4002
myrqlite-container-2  | [store] 2025/06/16 18:55:54 ensuring data directory exists at /rqlite/file/data
myrqlite-container-2  | [store] 2025/06/16 18:55:54 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-2  | [snapshot-store] 2025/06/16 18:55:54 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-2  | [store] 2025/06/16 18:55:54 0 preexisting snapshots present
myrqlite-container-2  | [store] 2025/06/16 18:55:54 raft log is 0 bytes at open, indexes are: first 0, last 0
myrqlite-container-2  | [db] 2025/06/16 18:55:54 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-2  | [rqlited] 2025/06/16 18:55:54 checking that supplied join addresses don't serve HTTP(S)
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-1:4002: store not open (did you forget to set -join-as?)
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-2:4002: no leader (did you forget to set -join-as?)
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:54 failed to join via node at myrqlite-service-3:4002: read protobuf length: read tcp 172.18.0.2:35386->172.18.0.4:4002: read: connection reset by peer (did you forget to set -join-as?)
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:54 failed to join cluster at [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002], after 1 attempt(s)
myrqlite-container-2  | [cluster-bootstrap] 2025/06/16 18:55:54 failed to notify all targets: [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002] (failed to notify node at myrqlite-service-1:4002: store not open, will retry)
myrqlite-container-2  | [raft] 2025/06/16 18:55:56 [WARN]  no known peers, aborting election
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-1:4002: no leader (did you forget to set -join-as?)
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-2:4002: no leader (did you forget to set -join-as?)
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:56 failed to join via node at myrqlite-service-3:4002: no leader (did you forget to set -join-as?)
myrqlite-container-2  | [cluster-join] 2025/06/16 18:55:56 failed to join cluster at [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002], after 1 attempt(s)
myrqlite-container-2  | [store] 2025/06/16 18:55:56 reached expected bootstrap count of 3, starting cluster bootstrap
myrqlite-container-2  | [store] 2025/06/16 18:55:56 cluster bootstrap successful, servers: [{Voter myrqlite-node-1 myrqlite-host-1:4002} {Voter myrqlite-node-3 myrqlite-host-3:4002} {Voter myrqlite-node-2 myrqlite-host-2:4002}]
myrqlite-container-2  | [cluster-bootstrap] 2025/06/16 18:55:56 succeeded notifying all targets: [myrqlite-service-1:4002 myrqlite-service-2:4002 myrqlite-service-3:4002]
myrqlite-container-2  | [store] 2025/06/16 18:55:56 node myrqlite-node-3 is now Leader
myrqlite-container-2  | [cluster-bootstrap] 2025/06/16 18:55:58 boot operation marked done
myrqlite-container-2  | [rqlited] 2025/06/16 18:55:58 node HTTP API available at http://myrqlite-host-2:4001
myrqlite-container-2  | [rqlited] 2025/06/16 18:55:58 connect using the command-line tool via 'rqlite -H myrqlite-host-2 -p 4001'
```


#### 7. Interact with rqlite using the command-line tool

The guide demonstrates how to connect to the rqlite CLI within a running container and interact with the database.
The example shows connecting to `myrqlite-host-3` via `myrqlite-container-1`, highlighting that any node can be used to perform operations as changes are replicated across the cluster.

```bash
$ docker exec -it myrqlite-container-1 rqlite -H myrqlite-host-3
```

```sql
Welcome to the rqlite CLI.
Enter ".help" for usage hints.
Connected to http://myrqlite-host-3:4001 running version v8.37.4
myrqlite-host-3:4001>
```


#### 8. Perform database operations

The example of creating a table and inserting data confirms that the cluster is functional and that data replication is occurring.

```sql
myrqlite-host-3:4001> .timer on
myrqlite-host-3:4001> CREATE TABLE foo (id INTEGER NOT NULL PRIMARY KEY, name TEXT)
1 row affected (0.000000 sec)
Run Time: 0.000000 seconds
myrqlite-host-3:4001> INSERT INTO foo(name) VALUES ('Fiona'), ('Philip'), ('Olivier')
3 rows affected (0.000000 sec)
Run Time: 0.000000 seconds
myrqlite-host-3:4001> SELECT * FROM foo
+----+---------+
| id | name    |
+----+---------+
| 1  | Fiona   |
+----+---------+
| 2  | Philip  |
+----+---------+
| 3  | Olivier |
+----+---------+
Run Time: 0.000114 seconds
myrqlite-host-3:4001> .exit
```


#### 9. Stop and clean up the service

This command gracefully stops and removes all containers, networks, and volumes defined in the `compose.yaml` file, ensuring a clean tear down of the environment.

```bash
$ docker compose down
```

```text
[+] Running 4/4
 ✔ Container myrqlite-container-1             Removed                                     0.7s
 ✔ Container myrqlite-container-2             Removed                                     0.5s
 ✔ Container myrqlite-container-3             Removed                                     0.9s
 ✔ Network rqliteautomaticclustering_default  Removed                                     0.4s
```


**References**

[Automatic clustering](https://rqlite.io/docs/clustering/automatic-clustering/)
