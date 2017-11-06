# BOSH release for redis-debian

This BOSH release and deployment manifest deploy a cluster of redis-debian.

## Usage

This repository includes base manifests and operator files. They can be used for initial deployments and subsequently used for updating your deployments:

```
export BOSH_ENVIRONMENT=<bosh-alias>
export BOSH_DEPLOYMENT=redis-debian
git clone https://github.com/cloudfoundry-community/redis-debian-boshrelease.git
bosh deploy redis-debian-boshrelease/manifests/redis-debian.yml
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
bosh deploy redis-debian-boshrelease/manifests/redis-debian.yml
```
