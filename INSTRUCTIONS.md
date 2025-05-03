# Ansible lint

Description on how to install ansible lint. Ansible has to be already installed, follow instructions [here](../1-install-ansible-pipx/INSTRUCTIONS.md).

## install lint

### Clean install

Install Both Into a Shared pipx Environment

```bash
pipx install ansible --include-deps
pipx inject ansible ansible-lint
```

### after ansible install

If ansible was already installed a long time ago, you can match existing version

```bash
pipx inject --include-apps ansible ansible-lint
```

if you know specific version

```bash
pipx install ansible-lint==6.22.1
```

### Check

`ansible` version

```bash
ansible --version
```

output:

```text
ansible [core 2.18.3]
  config file = /home/user/.ansible.cfg
  configured module search path = ['/home/user/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /home/user/.local/share/pipx/venvs/ansible/lib/python3.12/site-packages/ansible
  ansible collection location = /home/user/.ansible/collections:/usr/share/ansible/collections
  executable location = /home/user/.local/bin/ansible
  python version = 3.12.3 (main, Feb  4 2025, 14:48:35) [GCC 13.3.0] (/home/user/.local/share/pipx/venvs/ansible/bin/python)
  jinja version = 3.1.6
  libyaml = True
```

`ansible-lint` version

```bash
ansible-lint --version
```

output:

```text
ansible-lint 25.4.0 using ansible-core:2.18.3 ansible-compat:25.1.5 ruamel-yaml:0.18.10 ruamel-yaml-clib:0.2.12
WARNING  Project directory /.ansible cannot be used for caching as it is not writable.
WARNING  Using unique temporary directory /tmp/.ansible-0aaa for caching.
```

## Validate

check regular yaml file.

```bash
ansible-lint upload.yaml
```

output:

```text
WARNING  Listing 2 violation(s) that are fatal
risky-file-permissions: File permissions unset or incorrect.
upload.yaml:16 Task/Handler: Upload needed files for hands-on on Rebrain

yaml[empty-lines]: Too many blank lines (1 > 0)
upload.yaml:21

Read documentation for instructions on how to ignore specific rule violations.

# Rule Violation Summary

  1 yaml profile:basic tags:formatting,yaml
  1 risky-file-permissions profile:basic tags:unpredictability

Failed: 2 failure(s), 0 warning(s) on 1 files. Last profile that met the validation criteria was 'min'.
```

## Extras

### ansible config

you can add extra config for ansible roles, info in [chapter 10](../10-ansible-galaxy/THEORY.md). create a `ansible.cfg` config file.

```text
[defaults]
roles_path = ./ansible-roles:/etc/ansible/roles
```