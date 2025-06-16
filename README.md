# Docker Compose

Docker Compose is a tool for defining and running multi-container applications.
It is the key to unlocking a streamlined and efficient development and deployment experience.

These directions assume you've already got Docker Compose installed.
If not, head over to [docs.docker.com/compose](https://docs.docker.com/compose/) to learn how to get set up.

---

### rqlite Implementation Examples

This documentation provides examples for setting up rqlite using Docker Compose, covering the following scenarios:

  * **Single-node database**: A basic setup for a standalone rqlite instance.
  * **General clustering with 3 nodes**: A manual configuration for a three-node rqlite cluster.
  * **Automatic clustering with 3 nodes**: An example demonstrating automatic cluster formation with three rqlite nodes.


#### Conventions

To make `compose.yaml` files easier to understand, I use distinct and explicit naming conventions:

  * **name**: `rqlite<project>`
  * **services**: `myrqlite-service-<id>`
  * **container_name**: `myrqlite-container-<id>`
  * **hostname**: `myrqlite-host-<id>`
  * **volumes**: `./rqlite-data/myrqlite-node-<id>:/rqlite/file`.
For testing, we recommend creating a local `rqlite-data` folder using the command `mkdir -p rqlite-data`.
  * **NODE_ID**: `myrqlite-node-<id>`

The initial comment lines within each `compose.yaml` file describe the test versions at the time of publication.
However, the `latest` version can typically be used in the code.

```yaml
# Created: <YYYY>-<MM>-<DD> <hh>:<mm>:<ss>
# Updated: <YYYY>-<MM>-<DD> <hh>:<mm>:<ss>
# Language: Docker Compose version <version>
# Images:
#    - rqlite/rqlite:<version>
#    - <image>:<version>
# Project: <project name>
```

---


## [Single-node database](./rqliteSingleNode/)


## [General clustering with 3 nodes](./rqliteGeneralClustering/)


## [Automatic clustering with 3 nodes](./rqliteAutomaticClustering/)
