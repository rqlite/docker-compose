# Docker Compose


## Single-node database

Let's walk through setting up a single rqlite node using Docker Compose.


### Step-by-Step Setup


#### 1. Project directory `rqliteSingleNode`

Folder, `rqliteSingleNode` is where your `compose.yaml` file will reside.


#### 2. Review the `rqliteSingleNode` `compose.yaml` file

This file defines our single rqlite service.
Take a look at its contents:

```bash
$ cat compose.yaml
```

```yaml
# Created: 2025-06-13 23:47:19
# Updated: 2026-03-18 20:38:08
# Language: Docker Compose version v5.1.0
# Images:
#    - rqlite/rqlite:9.4.5
# Project: rqlite Single-Node

name: rqliteSingleNode

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

volumes:
    rqlite-data-node-1: {}
```


#### 3. Launch the rqlite service

Use `docker compose up -d` to build, create, start, and attach to the container in **detached mode**.
This means the container will run in the background.

```bash
$ docker compose up -d
```

```text
[+] up 12/12
 ✔ Image rqlite/rqlite:latest                 Pulled            5.0s
 ✔ Network rqlitesinglenode_default           Created           0.1s
 ✔ Volume rqlitesinglenode_rqlite-data-node-1 Created           0.0s
 ✔ Container myrqlite-container-1             Started           0.3s
```


#### 4. Verify the service status

Check that your rqlite container is running and healthy using `docker compose ps`.

```bash
$ docker compose ps
```

```text
NAME                   IMAGE                  COMMAND                  SERVICE              CREATED          STATUS                    PORTS
myrqlite-container-1   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-1   50 seconds ago   Up 50 seconds (healthy)   0.0.0.0:4001-4002->4001-4002/tcp, [::]:4001-4002->4001-4002/tcp
```


#### 5. Inspect the service logs

To see what's happening inside your rqlite container, check its logs.
This is useful for troubleshooting or confirming the service started correctly.

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
myrqlite-container-1  | [rqlited] 2026/03/18 20:06:09 rqlited starting, version v9.4.5, SQLite 3.51.2, commit 41d7a347ab2db0fa494fc9553b9d9467c343a678, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-1  | [rqlited] 2026/03/18 20:06:09 go1.26.1, target architecture is amd64, operating system target is linux
myrqlite-container-1  | [rqlited] 2026/03/18 20:06:09 launch command: /bin/rqlited -node-id myrqlite-node-1 -http-addr myrqlite-host-1:4001 -http-adv-addr myrqlite-host-1:4001 -raft-addr myrqlite-host-1:4002 -raft-adv-addr myrqlite-host-1:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip /rqlite/file/data
myrqlite-container-1  | [mux] 2026/03/18 20:06:09 mux serving on 172.20.0.2:4002, advertising myrqlite-host-1:4002
myrqlite-container-1  | [rqlited] 2026/03/18 20:06:09 no preexisting node state detected in /rqlite/file/data, node may be bootstrapping
myrqlite-container-1  | [cluster] 2026/03/18 20:06:09 service listening on myrqlite-host-1:4002
myrqlite-container-1  | [http] 2026/03/18 20:06:09 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-1  | [http] 2026/03/18 20:06:09 service listening on 172.20.0.2:4001
myrqlite-container-1  | [store] 2026/03/18 20:06:09 opening store with node ID myrqlite-node-1, listening on myrqlite-host-1:4002
myrqlite-container-1  | [store] 2026/03/18 20:06:09 ensuring data directory exists at /rqlite/file/data
myrqlite-container-1  | [store] 2026/03/18 20:06:09 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-1  | [snapshot-store] 2026/03/18 20:06:09 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-1  | [store] 2026/03/18 20:06:09 0 preexisting snapshots present
myrqlite-container-1  | [store] 2026/03/18 20:06:09 raft log is 0 bytes at open, no entries present
myrqlite-container-1  | [db] 2026/03/18 20:06:09 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-1  | [rqlited] 2026/03/18 20:06:09 bootstrapping single new node
myrqlite-container-1  | [rqlited] 2026/03/18 20:06:09 node HTTP API available at http://myrqlite-host-1:4001
myrqlite-container-1  | [rqlited] 2026/03/18 20:06:09 connect using the command-line tool via 'rqlite -H myrqlite-host-1 -p 4001'
myrqlite-container-1  | [raft] 2026/03/18 20:06:11 [WARN]  heartbeat timeout reached, starting election: last-leader-addr= last-leader-id=
myrqlite-container-1  | [store] 2026/03/18 20:06:11 this node (ID=myrqlite-node-1, addr=myrqlite-host-1:4002) is now Leader
```


#### 6. Interact with rqlite using the command-line tool

You can directly access the rqlite CLI inside the running container.

```bash
$ docker exec -it myrqlite-container-1 rqlite -H myrqlite-host-1
```

```sql
Welcome to the rqlite CLI.
Enter ".help" for usage hints.
Connected to http://myrqlite-host-1:4001 running version v9.4.5
myrqlite-host-1:4001>
```


#### 7. Perform database operations

Now that you're connected to the rqlite CLI, you can create tables, insert data, and query it just like you would with SQLite.

```sql
myrqlite-host-1:4001> .timer on
myrqlite-host-1:4001> CREATE TABLE foo (id INTEGER NOT NULL PRIMARY KEY, name TEXT)
0 row affected (0.000782 sec)
myrqlite-host-1:4001> INSERT INTO foo(name) VALUES ('Fiona'), ('Philip'), ('Olivier')
3 rows affected (0.000242 sec)
myrqlite-host-1:4001> SELECT * FROM foo
+----+---------+
| id | name    |
+----+---------+
| 1  | Fiona   |
+----+---------+
| 2  | Philip  |
+----+---------+
| 3  | Olivier |
+----+---------+
Run Time: 0.000236 seconds
myrqlite-host-1:4001> .exit
bye~
```


#### 8. Stop and clean up the service

When you're finished, `docker compose down` will stop the containers and remove the containers, networks, volumes, and images created by `docker compose up`.

```bash
$ docker compose down
```

```text
[+] down 2/2
 ✔ Container myrqlite-container-1   Removed                     0.4s
 ✔ Network rqlitesinglenode_default Removed                     0.3s
```
