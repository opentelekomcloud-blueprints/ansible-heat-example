# Example of using Ansible with Heat packed with SoftwareConfig

Templates may use SoftwareConfig and SoftwareDeployment resources. It gives a benefit of avality to perform runtime configuration of the servers without recreating them (as with using user_data). However this require specially prepared image. It can be done as follows:
```
  sudo pip install git+https://opendev.org/openstack/diskimage-builder.git
  git clone https://opendev.org/openstack/tripleo-image-elements.git
  git clone https://opendev.org/openstack/heat-agents.git
  export ELEMENTS_PATH=tripleo-image-elements/elements:heat-agents/
  disk-image-create vm \
    fedora selinux-permissive \
    os-collect-config \
    os-refresh-config \
    os-apply-config \
    heat-config \
    heat-config-ansible \
    heat-config-cfn-init \
    heat-config-docker-compose \
    heat-config-kubelet \
    heat-config-puppet \
    heat-config-salt \
    heat-config-script \
    -o fedora-software-config.qcow2
  openstack image create --disk-format qcow2 --container-format bare --min-disk 6 --min-ram 512 fedora-software-config -f fedora-software-config.qcow2
```

# Description

Ansible is used initially to prepare KeyPair and trigger the Heat stack
provisioning. It can be further used to provision some secrets to the started
instances, if those should not be visible in the Heat SoftwareConfigs (security
concern). When Heat stack is created, instance is started and the software
configurations are deployed. Those in turn might be used to enable a follow-up
cronjob with ansible-pull, which is periodically fetches "commands" to be
executed locally and thus is a small step forward 0-effort operations.

Execution of the playbook can be done with the following command (cloud
configuration should be done in advance in either `clouds.yaml` with properly
setting of the OS_CLOUD env variable, or all the way with only env):

```
  ansible-playbook -i inventory/testing playbooks/main.yaml
```
