
+++
title = "System tests"
type = "chapter"
weight = 3
+++

Here's how to execute system tests locally.

## Deploy a bare metal machine on your homelab

You know how to do it. Right? 

## Install LXD and prepare the environment

```
sudo apt-get update
sudo apt-get install tox

snap install yq
snap install lxd
lxd init --auto
```

## Execute the tests (if you have to re-execute the tests, repeat from here)

```
lxc delete maas-system-build --force || true; lxc delete maas-system-maas --force || true; lxc delete maas-client --force || true;
lxc list; pgrep -a qemu || true; rm -rf system-tests; git clone https://git.launchpad.net/~maas-committers/maas-ci/+git/system-tests --branch master --depth 10 system-tests; cd system-tests && git show --no-patch;

cd lxd_configs 
./configure_lxd.sh
./configure_lxd_key.sh
./generate_lxd_base_config.py > ../config.base.yaml 
cd ../
```

## Generate the config for the tests

```
export MAAS_REPO=https://git.launchpad.net/~r00ta/maas
export MAAS_BRANCH=fix-vault-integration
tox -e generate_config -- --hw-sync --deb --git-repo $MAAS_REPO  --git-branch $MAAS_BRANCH --ppa ppa:maas-committers/latest-deps --only-vms --vault --containers-image=ubuntu:24.04 --image-stream-url=http://images.maas.io/ephemeral-v3/stable/ config.base.yaml config.yaml image_mapping.yaml
```


## Run the tests

```
tox -e cog 
tox run -e env_builder -- --log-cli-level info | tee env_builder.out 
tox run -e general_tests -- --log-cli-level info 
TEST_ENVS=$(cat config.yaml | yq '.machines.vms.instances | keys | join(",")') 
tox run-parallel --parallel-no-spinner -p all -e ${TEST_ENVS} -- --log-cli-level info 
 ```
