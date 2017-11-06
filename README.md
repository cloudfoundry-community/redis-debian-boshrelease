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
