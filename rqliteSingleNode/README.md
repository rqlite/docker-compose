# Docker Compose


## Single-node database

Let's walk through setting up a single rqlite node using Docker Compose.


### Step-by-Step Setup


#### 1. Project directory `rqliteSingleNode`

Folder, `rqliteSingleNode` is where your `compose.yaml file will reside.


#### 2. Create the data persistence folder

rqlite needs a place to store its data.
We'll create a local folder named `rqlite-data` that will be mounted into the container.

```bash
$ mkdir -p rqlite-data
```


#### 3. Review the `rqliteSingleNode` `compose.yaml` file

This file defines our single rqlite service.
Take a look at its contents:

```bash
$ cat compose.yaml
```

```yaml
# Created: 2025-06-13 23:47:19
# Updated:
# Language: Docker Compose version v2.36.2
# Images:
#    - rqlite/rqlite:8.37.4
# Project: rqlite Single-Node

name: rqliteSingleNode

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
```


#### 4. Launch the rqlite service

Use `docker compose up -d` to build, create, start, and attach to the container in **detached mode**.
This means the container will run in the background.

```bash
$ docker compose up -d
```

```text
[+] Running 8/8
 ✔ myrqlite-service-1 Pulled                            6.2s
   ✔ fe07684b16b8 Already exists                        0.0s
   ✔ 0ab638900fe9 Pull complete                         1.7s
   ✔ ee0ca339bb46 Pull complete                         1.7s
   ✔ bbb0f06e65bb Pull complete                         3.0s
   ✔ af50c63557f2 Pull complete                         3.1s
   ✔ 36f51aa6517e Pull complete                         3.1s
   ✔ be1e3eb83208 Pull complete                         3.2s
[+] Running 2/2
 ✔ Network rqlitesinglenode_default  Created            0.0s
 ✔ Container myrqlite-container-1    Started            0.3s
```


#### 5. Verify the service status

Check that your rqlite container is running and healthy using `docker compose ps.

```bash
$ docker compose ps
```

```text
NAME                   IMAGE                  COMMAND                  SERVICE              CREATED         STATUS                   PORTS
myrqlite-container-1   rqlite/rqlite:latest   "docker-entrypoint.s…"   myrqlite-service-1   3 minutes ago   Up 3 minutes (healthy)   0.0.0.0:4001-4002->4001-4002/tcp, [::]:4001-4002->4001-4002/tcp
```


#### 6. Inspect the service logs

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
myrqlite-container-1  | [rqlited] 2025/06/13 22:22:02 rqlited starting, version v8.37.4, SQLite 3.49.1, commit 4870c35c694f3173b83c45a275d7a943c8187535, branch master, compiler (toolchain) gc, compiler (command) musl-gcc
myrqlite-container-1  | [rqlited] 2025/06/13 22:22:02 go1.24.4, target architecture is amd64, operating system target is linux
myrqlite-container-1  | [rqlited] 2025/06/13 22:22:02 launch command: /bin/rqlited -node-id myrqlite-node-1 -http-addr myrqlite-host-1:4001 -http-adv-addr myrqlite-host-1:4001 -raft-addr myrqlite-host-1:4002 -raft-adv-addr myrqlite-host-1:4002 -extensions-path=/opt/extensions/sqlean.zip,/opt/extensions/sqlite-vec.zip,/opt/extensions/misc.zip /rqlite/file/data
myrqlite-container-1  | [mux] 2025/06/13 22:22:02 mux serving on 172.18.0.2:4002, advertising myrqlite-host-1:4002
myrqlite-container-1  | [rqlited] 2025/06/13 22:22:02 preexisting node state detected in /rqlite/file/data
myrqlite-container-1  | [cluster] 2025/06/13 22:22:02 service listening on myrqlite-host-1:4002
myrqlite-container-1  | [http] 2025/06/13 22:22:02 execute queue processing started with capacity 1024, batch size 128, timeout 50ms
myrqlite-container-1  | [http] 2025/06/13 22:22:02 service listening on 172.18.0.2:4001
myrqlite-container-1  | [store] 2025/06/13 22:22:02 opening store with node ID myrqlite-node-1, listening on myrqlite-host-1:4002
myrqlite-container-1  | [store] 2025/06/13 22:22:02 ensuring data directory exists at /rqlite/file/data
myrqlite-container-1  | [store] 2025/06/13 22:22:02 old v7 snapshot directory does not exist at /rqlite/file/data/snapshots, nothing to upgrade
myrqlite-container-1  | [snapshot-store] 2025/06/13 22:22:02 store initialized using /rqlite/file/data/rsnapshots
myrqlite-container-1  | [store] 2025/06/13 22:22:02 0 preexisting snapshots present
myrqlite-container-1  | [store] 2025/06/13 22:22:02 raft log is 32768 bytes at open, indexes are: first 1, last 7
myrqlite-container-1  | [db] 2025/06/13 22:22:02 loaded extensions: amatch.so, anycollseq.so, base64.so, base85.so, basexx.so, btreeinfo.so, carray.so, cksumvfs.so, closure.so, completion.so, compress.so, decimal.so, eval.so, explain.so, fuzzer.so, ieee754.so, memstat.so, nextchar.so, noop.so, percentile.so, prefixes.so, qpvtab.so, randomjson.so, regexp.so, remember.so, rot13.so, series.so, sha1.so, shathree.so, spellfix.so, sqlar.so, sqlean.so, stmt.so, stmtrand.so, templatevtab.so, totype.so, uint.so, unionvtab.so, urifuncs.so, uuid.so, vec0.so, vtablog.so, vtshim.so, wholenumber.so, zorder.so
myrqlite-container-1  | [rqlited] 2025/06/13 22:22:02 node HTTP API available at http://myrqlite-host-1:4001
myrqlite-container-1  | [rqlited] 2025/06/13 22:22:02 connect using the command-line tool via 'rqlite -H myrqlite-host-1 -p 4001'
myrqlite-container-1  | [raft] 2025/06/13 22:22:03 [WARN]  heartbeat timeout reached, starting election: last-leader-addr= last-leader-id=
myrqlite-container-1  | [store] 2025/06/13 22:22:03 this node (ID=myrqlite-node-1) is now Leader
myrqlite-container-1  | [store] 2025/06/13 22:22:03 first log applied since node start, log at index 3
```


#### 7. Interact with rqlite using the command-line tool

You can directly access the rqlite CLI inside the running container.

```bash
$ docker exec -it myrqlite-container-1 rqlite -H myrqlite-host-1
```

```sql
Welcome to the rqlite CLI.
Enter ".help" for usage hints.
Connected to http://myrqlite-host-1:4001 running version v8.37.4
myrqlite-host-1:4001>
```


#### 8. Perform database operations

Now that you're connected to the rqlite CLI, you can create tables, insert data, and query it just like you would with SQLite.

```sql
myrqlite-host-1:4001> .timer on
myrqlite-host-1:4001> CREATE TABLE foo (id INTEGER NOT NULL PRIMARY KEY, name TEXT)
1 row affected (0.000000 sec)
Run Time: 0.000000 seconds
myrqlite-host-1:4001> INSERT INTO foo(name) VALUES ('Fiona'), ('Philip'), ('Olivier')
3 rows affected (0.000000 sec)
Run Time: 0.000000 seconds
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
Run Time: 0.000343 seconds
myrqlite-host-1:4001> .exit
```


#### 9. Stop and clean up the service

When you're finished, docker compose down will stop the containers and remove the containers, networks, volumes, and images created by docker compose up.

```bash
$ docker compose down
```

```text
[+] Running 2/2
 ✔ Container myrqlite-container-1    Removed            0.4s
 ✔ Network rqlitesinglenode_default  Removed            0.4s
```
