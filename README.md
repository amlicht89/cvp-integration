# Ansible & CVP Integration Framework

This repo focuses on the basic interactions between CVP and Ansible using the Arista AVD/CVP Collections.  It allows the user flexibility in managing device configurations with CVP and Ansible.  Configlets make up a device's full configuration.  You can manage the configlets inside of CVP or generate them dynamically with Ansible and merge them inside of CVP.

![CVP Ansible Image](images/CVP-Ansible%20Integration.png)
## Requirements

- Python 3 & Ansible 2.9+
- Clone Arista AVD and CVP collections (instructions below)
- Update `ansible.cfg` with collections_path

## Recommendation

Use a Docker Container with all necessary modules installed.  Consider building a docker image and container from the instructions found here: <https://github.com/mthiel117/dockerbuild>.

Once the image is built run the container and attach it a volume to local directory and location of Arista collections.  Sample Docker Run below.  Turn this into a alias for simple launching.

```bash
docker run -it --rm -v $(PWD):/projects -v ~/git_projects/arista-ansible:/collections mycentosbox/base
```



## Clone Arista AVD & CVP Collections

```bash
# Create a directory for collections
mkdir collections
cd collections

# Clone Arista AVD & CVP collections
git clone https://github.com/aristanetworks/ansible-avd.git
git clone https://github.com/aristanetworks/ansible-cvp.git

```

Update `ansible.cfg` with the path to the Arista AVD & CVP collections

```bash
[defaults]
collections_paths = /collections/ansible-cvp:/collections/ansible-avd/
```

## Data Inputs

There are a few files that need to be updated to provide data input for the playbooks.

- `inventory.yml` - Update CVP host with ip or fqdn of your CVP system
- `datafiles/svis.yml`  - Dictionary for SVI's to be applied to Devices
- `datafiles/devices.csv` - List of Devices (CSV format) used to create base configlets
- `group_vars/CVP_SERVERS/authconnection.yml` - CVP login and password and connection info
- `group_vars/CVP_SERVERS/containers.yml` - Dictionary of Container Topology
- `group_vars/CVP_SERVERS/devices.yml` - Dictionary of Devices, Containers, Configlets

## Playbooks

There are 2 main playbooks that we will focus on.

- `playbooks/generate_configs.yml` - Generates Base configlets for each device and Common SVIs
- `playbooks/update_cvp.yml` - Updates Continer Topology & Uploads configlets into CVP and attaches them to the appropriate device and/or container.  Tags (`upload,update,full`) are used to control which tasks are executed in playbook

  - `upload` tag will execute task to upload configlet to CVP
  - `update` tag will execute tasks to move devices, attach configlets and create container topology
  - `full` tag will execute all tasks above

### Running playbooks

```bash
# Generate Configs
ansible-playbook playbooks/generate_configs.yml

# Upload Configlets
ansible-playbook playbooks/update_cvp.yml --tags upload

# Update Device configlets and Container Topology
ansible-playbook playbooks/update_cvp.yml --tags update

# Do Everything: Upload Configlets and Update Device configlets and Container Topology
ansible-playbook playbooks/update_cvp.yml --tags full
```
