# Docker Compose


## General clustering with 3 nodes

Let's walk through setting up a manually configured three-node rqlite cluster using Docker Compose.
This setup involves explicitly instructing each joining node to connect to an existing node.


### Step-by-Step Setup


#### 1. Project directory `rqliteGeneralClustering`

Folder, `rqliteGeneralClustering` is where your `compose.yaml`file will reside.


#### 2. Review the `rqliteGeneralClustering` `compose.yaml` file

This file defines our three rqlite services, `myrqlite-service-1`, `myrqlite-service-2`, and `myrqlite-service-3`.
Notice how `myrqlite-service-2` and `myrqlite-service-3` use the `-join` command to connect to `myrqlite-service-1`.
Take a look at its contents:

```bash
$ cat compose.yaml
```

```yaml
# Created: 2025-06-03 21:13:24
# Updated: 2026-03-18 21:17:08
# Language: Docker Compose version v5.1.0
# Images:
#   - rqlite/rqlite:9.4.5
# Project: rqlite General Clustering

name: rqliteCluster

services:

  myrqlite-service-1:
    image: rqlite/rqlite:latest
    container_name: myrqlite-container-1
    hostname: myrqlite-host-1
    volumes:
      - rqlite-data-node-1:/rqlite/file
    ports:
      - "4001:4001"
      - "4002:4002"
    environment:
      NODE_ID: myrqlite-node-1
      HTTP_ADDR: myrqlite-host-1:4001
      RAFT_ADDR: myrqlite-host-1:4002
      SQLITE_EXTENSIONS: "sqlean,sqlite-vec,misc"
    healthcheck:
      test: ["CMD", "wget", "-q", "-O", "/dev/null", "http://myrqlite-host-1:4001/status"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  myrqlite-service-2:
    image: rqlite/rqlite:latest
    container_name: myrqlite-container-2
    hostname: myrqlite-host-2
    volumes:
      - rqlite-data-node-2:/rqlite/file
    ports:
      - "4011:4001"
      - "4012:4002"
    command: -join myrqlite-service-1:4002
    environment:
      NODE_ID: myrqlite-node-2
      HTTP_ADDR: myrqlite-host-2:4001
      RAFT_ADDR: myrqlite-host-2:4002
      SQLITE_EXTENSIONS: "sqlean,sqlite-vec,misc"
    depends_on:
      myrqlite-service-1:
        condition: service_healthy

  myrqlite-service-3:
    image: rqlite/rqlite:latest
    container_name: myrqlite-container-3
    hostname: myrqlite-host-3
    volumes:
      - rqlite-data-node-3:/rqlite/file
    ports:
      - "4021:4001"
      - "4022:4002"
    command: -join myrqlite-service-1:4002
    environment:
      NODE_ID: myrqlite-node-3
      HTTP_ADDR: myrqlite-host-3:4001
      RAFT_ADDR: myrqlite-host-3:4002
      SQLITE_EXTENSIONS: "sqlean,sqlite-vec,misc"
    depends_on:
      myrqlite-service-1:
        condition: service_healthy

volumes:
    rqlite-data-node-1: {}
    rqlite-data-node-2: {}
    rqlite-data-node-3: {}
```


#### 3. Launch the rqlite service

Use `docker compose up -d` to build, create, start, and attach to the containers in **detached mode**.
This means the cluster will run in the background.

```bash
$ docker compose up -d
```

```text
[+] up 16/16
 ✔ Image rqlite/rqlite:latest              Pulled              10.9s
 ✔ Network rqlitecluster_default           Created             0.1s
 ✔ Volume rqlitecluster_rqlite-data-node-2 Created             0.0s
 ✔ Volume rqlitecluster_rqlite-data-node-3 Created             0.0s
 ✔ Volume rqlitecluster_rqlite-data-node-1 Created             0.0s
 ✔ Container myrqlite-container-1          Healthy             6.0s
 ✔ Container myrqlite-container-3          Started             6.1s
 ✔ Container myrqlite-container-2          Started             6.1s
```


#### 4. Verify the service status

Check that your rqlite containers are running and healthy using `docker compose ps`.
You should see all three services listed.

```bash
$ docker compose ps
```

```text
NAME                   IMAGE                  COMMAND                  SERVICE              CREATED          STATUS                    PORTS
myrqlite-container-1   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-1   29 seconds ago   Up 28 seconds (healthy)   0.0.0.0:4001-4002->4001-4002/tcp, [::]:4001-4002->4001-4002/tcp
myrqlite-container-2   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-2   29 seconds ago   Up 22 seconds             0.0.0.0:4011->4001/tcp, [::]:4011->4001/tcp, 0.0.0.0:4012->4002/tcp, [::]:4012->4002/tcp
myrqlite-container-3   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-3   29 seconds ago   Up 22 seconds             0.0.0.0:4021->4001/tcp, [::]:4021->4001/tcp, 0.0.0.0:4022->4002/tcp, [::]:4022->4002/tcp
```


#### 5. Inspect the cluster logs

To see what's happening across your rqlite cluster, check the combined logs.
This is useful for troubleshooting or confirming the cluster formed correctly.

```bash
$ docker compose logs
```

```text
myrqlite-container-3  |
myrqlite-container-3  |             _ _ _
myrqlite-container-3  |            | (_) |
myrqlite-container-3  |   _ __ __ _| |_| |_ ___
myrqlite-container-3  |  | '__/ _  | | | __/ _ \   The lightweight, distributed
myrqlite-container-3  |  | | | (_| | | | ||  __/   relational database.
myrqlite-container-3  |  |_|  \__, |_|_|\__\___|
myrqlite-container-3  |          | |               www.rqlite.io
myrqlite-container-3  |          |_|
myrqlite-container-2  |
myrqlite-container-2  |             _ _ _
myrqlite-container-2  |            | (_) |
myrqlite-container-2  |   _ __ __ _| |_| |_ ___
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 rqlited starting, version v9.4.5, SQLite 3.51.2, commit 41d7a347ab2db0fa494fc9553b9d9467c343a678, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 go1.26.1, target architecture is amd64, operating system target is linux
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 launch command: /bin/rqlited -node-id myrqlite-node-2 -http-addr myrqlite-host-2:4001 -http-adv-addr myrqlite-host-2:4001 -raft-addr myrqlite-host-2:4002 -raft-adv-addr myrqlite-host-2:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip -join myrqlite-service-1:4002 /rqlite/file/data
myrqlite-container-2  |  | '__/ _  | | | __/ _ \   The lightweight, distributed
myrqlite-container-2  |  | | | (_| | | | ||  __/   relational database.
myrqlite-container-2  |  |_|  \__, |_|_|\__\___|
myrqlite-container-2  |          | |               www.rqlite.io
myrqlite-container-3  |
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 rqlited starting, version v9.4.5, SQLite 3.51.2, commit 41d7a347ab2db0fa494fc9553b9d9467c343a678, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 go1.26.1, target architecture is amd64, operating system target is linux
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 launch command: /bin/rqlited -node-id myrqlite-node-3 -http-addr myrqlite-host-3:4001 -http-adv-addr myrqlite-host-3:4001 -raft-addr myrqlite-host-3:4002 -raft-adv-addr myrqlite-host-3:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip -join myrqlite-service-1:4002 /rqlite/file/data
myrqlite-container-3  | [mux] 2026/03/18 20:26:03 mux serving on 172.20.0.4:4002, advertising myrqlite-host-3:4002
myrqlite-container-1  |
myrqlite-container-1  |             _ _ _
myrqlite-container-1  |            | (_) |
myrqlite-container-1  |   _ __ __ _| |_| |_ ___
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 no preexisting node state detected in /rqlite/file/data, node may be bootstrapping
myrqlite-container-1  |  | '__/ _  | | | __/ _ \   The lightweight, distributed
myrqlite-container-1  |  | | | (_| | | | ||  __/   relational database.
myrqlite-container-2  |          |_|
myrqlite-container-3  | [cluster] 2026/03/18 20:26:03 service listening on myrqlite-host-3:4002
myrqlite-container-2  |
myrqlite-container-1  |  |_|  \__, |_|_|\__\___|
myrqlite-container-2  | [mux] 2026/03/18 20:26:03 mux serving on 172.20.0.3:4002, advertising myrqlite-host-2:4002
myrqlite-container-1  |          | |               www.rqlite.io
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 no preexisting node state detected in /rqlite/file/data, node may be bootstrapping
myrqlite-container-3  | [http] 2026/03/18 20:26:03 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-2  | [cluster] 2026/03/18 20:26:03 service listening on myrqlite-host-2:4002
myrqlite-container-3  | [http] 2026/03/18 20:26:03 service listening on 172.20.0.4:4001
myrqlite-container-3  | [store] 2026/03/18 20:26:03 opening store with node ID myrqlite-node-3, listening on myrqlite-host-3:4002
myrqlite-container-3  | [store] 2026/03/18 20:26:03 ensuring data directory exists at /rqlite/file/data
myrqlite-container-1  |          |_|
myrqlite-container-1  |
myrqlite-container-1  | [rqlited] 2026/03/18 20:25:57 rqlited starting, version v9.4.5, SQLite 3.51.2, commit 41d7a347ab2db0fa494fc9553b9d9467c343a678, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-1  | [rqlited] 2026/03/18 20:25:57 go1.26.1, target architecture is amd64, operating system target is linux
myrqlite-container-1  | [rqlited] 2026/03/18 20:25:57 launch command: /bin/rqlited -node-id myrqlite-node-1 -http-addr myrqlite-host-1:4001 -http-adv-addr myrqlite-host-1:4001 -raft-addr myrqlite-host-1:4002 -raft-adv-addr myrqlite-host-1:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip /rqlite/file/data
myrqlite-container-1  | [mux] 2026/03/18 20:25:57 mux serving on 172.20.0.2:4002, advertising myrqlite-host-1:4002
myrqlite-container-1  | [rqlited] 2026/03/18 20:25:57 no preexisting node state detected in /rqlite/file/data, node may be bootstrapping
myrqlite-container-3  | [store] 2026/03/18 20:26:03 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-3  | [snapshot-store] 2026/03/18 20:26:03 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-3  | [store] 2026/03/18 20:26:03 0 preexisting snapshots present
myrqlite-container-3  | [store] 2026/03/18 20:26:03 raft log is 0 bytes at open, no entries present
myrqlite-container-3  | [db] 2026/03/18 20:26:03 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 checking that supplied join addresses don't serve HTTP(S)
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 successfully joined cluster at myrqlite-service-1:4002
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 node HTTP API available at http://myrqlite-host-3:4001
myrqlite-container-3  | [rqlited] 2026/03/18 20:26:03 connect using the command-line tool via 'rqlite -H myrqlite-host-3 -p 4001'
myrqlite-container-3  | [raft] 2026/03/18 20:26:03 [WARN]  failed to get previous log: previous-index=4 last-index=0 error="log not found"
myrqlite-container-3  | [store] 2026/03/18 20:26:03 node myrqlite-node-1 at myrqlite-host-1:4002 is now Leader
myrqlite-container-1  | [cluster] 2026/03/18 20:25:57 service listening on myrqlite-host-1:4002
myrqlite-container-2  | [http] 2026/03/18 20:26:03 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-1  | [http] 2026/03/18 20:25:57 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-1  | [http] 2026/03/18 20:25:57 service listening on 172.20.0.2:4001
myrqlite-container-1  | [store] 2026/03/18 20:25:57 opening store with node ID myrqlite-node-1, listening on myrqlite-host-1:4002
myrqlite-container-1  | [store] 2026/03/18 20:25:57 ensuring data directory exists at /rqlite/file/data
myrqlite-container-1  | [store] 2026/03/18 20:25:57 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-1  | [snapshot-store] 2026/03/18 20:25:57 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-1  | [store] 2026/03/18 20:25:57 0 preexisting snapshots present
myrqlite-container-2  | [http] 2026/03/18 20:26:03 service listening on 172.20.0.3:4001
myrqlite-container-2  | [store] 2026/03/18 20:26:03 opening store with node ID myrqlite-node-2, listening on myrqlite-host-2:4002
myrqlite-container-2  | [store] 2026/03/18 20:26:03 ensuring data directory exists at /rqlite/file/data
myrqlite-container-2  | [store] 2026/03/18 20:26:03 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-2  | [snapshot-store] 2026/03/18 20:26:03 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-2  | [store] 2026/03/18 20:26:03 0 preexisting snapshots present
myrqlite-container-2  | [store] 2026/03/18 20:26:03 raft log is 0 bytes at open, no entries present
myrqlite-container-2  | [db] 2026/03/18 20:26:03 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 checking that supplied join addresses don't serve HTTP(S)
myrqlite-container-1  | [store] 2026/03/18 20:25:57 raft log is 0 bytes at open, no entries present
myrqlite-container-1  | [db] 2026/03/18 20:25:57 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-1  | [rqlited] 2026/03/18 20:25:57 bootstrapping single new node
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 successfully joined cluster at myrqlite-service-1:4002
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 node HTTP API available at http://myrqlite-host-2:4001
myrqlite-container-2  | [rqlited] 2026/03/18 20:26:03 connect using the command-line tool via 'rqlite -H myrqlite-host-2 -p 4001'
myrqlite-container-2  | [raft] 2026/03/18 20:26:03 [WARN]  failed to get previous log: previous-index=3 last-index=0 error="log not found"
myrqlite-container-2  | [store] 2026/03/18 20:26:03 node myrqlite-node-1 at myrqlite-host-1:4002 is now Leader
myrqlite-container-1  | [rqlited] 2026/03/18 20:25:57 node HTTP API available at http://myrqlite-host-1:4001
myrqlite-container-1  | [rqlited] 2026/03/18 20:25:57 connect using the command-line tool via 'rqlite -H myrqlite-host-1 -p 4001'
myrqlite-container-1  | [raft] 2026/03/18 20:25:59 [WARN]  heartbeat timeout reached, starting election: last-leader-addr= last-leader-id=
myrqlite-container-1  | [store] 2026/03/18 20:25:59 this node (ID=myrqlite-node-1, addr=myrqlite-host-1:4002) is now Leader
myrqlite-container-1  | [store] 2026/03/18 20:26:03 node with ID myrqlite-node-2, at myrqlite-host-2:4002, joined successfully as voter
myrqlite-container-1  | [raft] 2026/03/18 20:26:03 [WARN]  appendEntries rejected, sending older logs: peer="{Voter myrqlite-node-2 myrqlite-host-2:4002}" next=1
myrqlite-container-1  | [store] 2026/03/18 20:26:03 node with ID myrqlite-node-3, at myrqlite-host-3:4002, joined successfully as voter
myrqlite-container-1  | [raft] 2026/03/18 20:26:03 [WARN]  appendEntries rejected, sending older logs: peer="{Voter myrqlite-node-3 myrqlite-host-3:4002}" next=1
```


#### 6. Interact with rqlite using the command-line tool

You can directly access the rqlite CLI inside any of the running containers, connecting to any `myrqlite-host-<id>`.
For example, let's connect to `myrqlite-host-3` via `myrqlite-container-1`:

```bash
$ docker exec -it myrqlite-container-1 rqlite -H myrqlite-host-3
```

```sql
Welcome to the rqlite CLI.
Enter ".help" for usage hints.
Connected to http://myrqlite-host-3:4001 running version v9.4.5
myrqlite-host-3:4001>
```


#### 7. Perform database operations

Now that you're connected to the rqlite CLI, you can create tables, insert data, and query it just like you would with SQLite.
Changes made on one node will be replicated across the cluster.

```sql
myrqlite-host-3:4001> .timer on
myrqlite-host-3:4001> CREATE TABLE foo (id INTEGER NOT NULL PRIMARY KEY, name TEXT)
0 row affected (0.000525 sec)
myrqlite-host-3:4001> INSERT INTO foo(name) VALUES ('Fiona'), ('Philip'), ('Olivier')
3 rows affected (0.000309 sec)
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
Run Time: 0.000218 seconds
myrqlite-host-3:4001> .exit
bye~
```


#### 8. Stop and clean up the service

When you're finished, `docker compose down` will stop the containers and remove the containers, networks, volumes, and images created by `docker compose up`.

```bash
$ docker compose down
```

```text
[+] down 4/4
 ✔ Container myrqlite-container-2 Removed                      0.8ss
 ✔ Container myrqlite-container-3 Removed                      0.6ss
 ✔ Container myrqlite-container-1 Removed                      10.3s
 ✔ Network rqlitecluster_default  Removed                      0.4s
```

---

**References**

[Clustering general guidelines](https://rqlite.io/docs/clustering/general-guidelines/)
