# Proxmox homelab


## Dependencies
- [Mozilla SOPS](https://github.com/getsops/sops)
- [Age](https://github.com/FiloSottile/age)
- [Pre-commit](https://pre-commit.com)
- [Pre-commit hooks versions](https://github.com/pre-commit/pre-commit-hooks/tags)

SOPs and Age are used to encrypt variable. These should be in place on an local machine or remote runner executing the playbook.

Pre commit is also used for formatting and linting  within the repository.

### Pre commit

Installed via python
``` shell
pip3 install pre-commit
```

Upgrade ansible lint
``` shell
pip3 install --user --upgrade ansible-lint
```

Enable in the repo via
``` shell
pre-commit install
```

To test a run of the hooks located in the `.pre-commit-config.yaml` run
``` shell
pre-commit run -a
```

Once setup and enabled it will run automatically when committing via git.

### Age key

An Age key was generated via `age-keygen -o age.key` and is located in the root of this repo **BUT** it is not committed to version control and is excluded via the `.gitignore` file.


### Variable encryption

Retrieve the age key from the age.key file and run the following command.

``` bash
sops --encrypt --age xxxxxxxxxxxxxxxxxxxxxxxxxxx vars.yaml > vars.enc.yaml
```

This will take the unencrypted vars file and encrypt it outputting to a new file `vars.enc.yaml`. This file is then used via setting a variable sops and using Ansible lookup functionality in conjunction with

- community.sops.sops
- ansible.builtin.from_yaml

allows retrieving the values form the encrypted file and then using them in the playbook. PLease note that when referencing them int he playbook a prefix of sops will need to be used e.g. `sops.varName`.

The key-file location in the repository is defined in the `ansible.cfg` as per the `community.sops.sops` [age_keyfile](https://docs.ansible.com/ansible/latest/collections/community/sops/sops_vars.html#parameter-age_keyfile) parameter.
We also specify the SOPS binary location the `ansible.cfg` as per the `community.sops.sops` [binary](https://docs.ansible.com/ansible/latest/collections/community/sops/sops_vars.html#parameter-sops_binary) parameter.

## Playbook

The playbook is currently split into 3 roles:

- update
- containers
- vms

These logical groupings should cover activities carried out in each of the roles.

The update section is focused on updating the underlying OS used by Proxmox.

The remaining roles, containers and vms are targeted at managing the various Proxmox constructs fort hose two areas within proxmox itself.


### Proxmox credentials

See the encrypted vars files but re using some details that were manually setup for a Terraform user. May need to re-visit this and have Ansible actually provision a user and a role to be used in this playbook. Split the auth setup from the actual VM & container management.

The commands to setup the user and the role and bind the roe to the user are below for reference:

``` shell
pveum role add TerraformProv -privs "Datastore.AllocateSpace Datastore.Audit Pool.Allocate Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.Monitor VM.PowerMgmt SDN.Use"
pveum user add terraform-prov@pve --password xxxxxxxxxxxxxxxxxxxxxx
pveum aclmod / -user terraform-prov@pve -role TerraformProv

```

The password has been redacted above but can be viewed in the encrypted vars file, `vars.enc.yaml`, or in the unencrypted `vars.yaml`.
