# Demonstration of using Debian packages

This BOSH release deploys the Ubuntu Trusty package [redis-server](https://packages.ubuntu.com/trusty/redis-server), rather than using BOSH packaging.

This is a demonstration of the ability to use external packaging systems, such as Debian packages and remote Debian repositories, within BOSH deployments.

The installation of `redis-server` is described in the deployment manifest `manifests/redis-debian.yml` using the `prepare_env` job from `os-conf`:

```yaml
- name: prepare_env
  release: os-conf
  properties:
    script: |-
      #!/bin/bash

      apt-get update
      apt-get install redis-server -y

      /usr/bin/redis-server -v
```

The configuration of `redis-server` and the runtime management is performed by the job template in folder `jobs/redis-debian`. We use `bpm` to run `/usr/bin/redis-server` within a container, and the job template renders the `redis.conf` with properties provided by the deployment manifest:

```yaml
---
processes:
- name: redis-debian
  executable: /usr/bin/redis-server
  args: [/var/vcap/jobs/redis-debian/config/redis.conf]
  ...
```

The deployment manifest will deploy a two-node cluster:

```
$ bosh -d redis-debian instances
Instance                                           Process State  AZ  IPs
redis-debian/b2ba3dab-531f-40a0-b00d-de3805786d54  running        z1  10.244.0.141
redis-debian/f52ae425-389e-410b-9264-0c1171d4fae7  running        z2  10.244.0.142
```

If you `bosh ssh redis-debian/1` and tail the logs you will see that it forms a master-slave cluster:

```
==> /var/vcap/sys/log/redis-debian/redis.log <==
[1] 06 Nov 03:35:18.800 * Connecting to MASTER 10.244.0.141:6379
[1] 06 Nov 03:35:18.800 * MASTER <-> SLAVE sync started
[1] 06 Nov 03:35:18.800 * Non blocking connect for SYNC fired the event.
[1] 06 Nov 03:35:18.800 * Master replied to PING, replication can continue...
[1] 06 Nov 03:35:18.800 * Partial resynchronization not possible (no cached master)
[1] 06 Nov 03:35:18.814 * Full resync from master: 9790a38fe3ee146d9ad51800cb0c92abb11d89b2:1
[1] 06 Nov 03:35:18.921 * MASTER <-> SLAVE sync: receiving 18 bytes from master
[1] 06 Nov 03:35:18.921 * MASTER <-> SLAVE sync: Flushing old data
[1] 06 Nov 03:35:18.921 * MASTER <-> SLAVE sync: Loading DB in memory
[1] 06 Nov 03:35:18.921 * MASTER <-> SLAVE sync: Finished with success
```

## Usage

This repository includes base manifests and operator files. They can be used for initial deployments and subsequently used for updating your deployments:

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=redis-debian
git clone https://github.com/cloudfoundry-community/redis-debian-boshrelease.git
bosh deploy redis-debian-boshrelease/manifests/redis-debian.yml \
  -o <(./redis-debian-boshrelease/manifests/operators/pick-from-cloud-config.sh)
```

If your BOSH does not have Credhub/Config Server, then remember `--vars-store` to allow generation of passwords and certificates.

### Update

When new versions of `redis-debian-boshrelease` are released the `manifests/redis-debian.yml` file will be updated. This means you can easily `git pull` and `bosh deploy` to upgrade.

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=redis-debian
cd redis-debian-boshrelease
git pull
cd -
bosh deploy redis-debian-boshrelease/manifests/redis-debian.yml \
  -o <(./redis-debian-boshrelease/manifests/operators/pick-from-cloud-config.sh)
```
