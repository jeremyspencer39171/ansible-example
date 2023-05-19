# ansible-example


## Table of Contents

1. [Getting started](#start)

1.1. [Setting up the Ansible host VM](#setup)

1.2. [Running Ansible](#run_ansible)

2. [Explanation of the file structure and syntax](#structure)

2.1. [Top level](#top_level)

2.2. [Roles directory](#roles)

2.3. [Examples](#examples)

3. [Linting](#linting)


### Getting started <a name="start"></a>

- You will need a linux computer/VM to run Ansible from.
`It does not run on Windows`.
- An IDE is recommended, something with syntax highlighting will be very useful (e.g. `vscode` or `atom`)

**Before writing configuration in Ansible:**
Make sure you know how install and configure your software in the workspace.
It is much easier to write automated configuration tasks when you already know how to manually configure the workspace.

If something in here doesn't work, or is out of date, or you want something adding, just let us know and we'll update it.


#### Setting up the Ansible host VM: <a name="setup"></a>
Install Git and Ansible (if not already installed):
```sh
sudo yum install -y git ansible
```
**Note:** Ansible can also be installed via `pip`, and may give a more recent version:
```sh
pip install ansible
```
Clone this repository:
```sh
git clone REPOSITORY
cd ansible-example
```
There are multiple types of inventory file here these tell Ansible which hosts you want to configure.
`simple_inventory.txt` will contain something like:
```ini
[role_name]
my_host.domain.ac.uk
```
Where `role_name` would be the section of Ansible you are developing, for example here we have `example`.
```ini
[example]
my_host.domain.ac.uk
```
If you have other playbooks defined in either `site.yml` or an included playbook (like `playbooks/my_playbook.yml`) then you can use those instead.
You can define more than one here too, like:
```ini
[example]
my_host.domain.ac.uk

[example_two]
my_other_host.domain.ac.uk
```

Copy the private key you are using for your Ansible work to your VM as `ansible.key`.
We will assume you put these in your home directory.

##### Dynamic Inventory
You can run a script to generate an ansible inventory if you prefer, this can be useful if your hosts change over time.
For example if you have your hosts and roles stored in a database, you could run a script to fetch those and construct a json output.
For that you'd just use something like the example `dynamic_inventory.sh` instead of `simple_inventory.txt`.
`dynamic_inventory_source.json` shows the sort of information Ansible needs (you can also pass `hostvars` in this way).

#### Running Ansible <a name="run_ansible"></a>
Once you have written your Ansible tasks, you will want to run `ansible-playbook` to apply your changes to your target host.
Choose your target host and paste its hostname into `simple_inventory.txt`, specifying the role you want to configure.

To tell Ansible to configure your workspace, you'll need to run `ansible-playbook` from the this directory:
```sh
/usr/bin/ansible-playbook -i ~/simple_inventory.txt site.yml --private-key ~/ansible.key
```
Where `~/inventory.txt` is your inventory file, and `~/ansible.key` is the private key associated with the public key you have on the target host.
If you need to specify any variables to pass in, this should be done via the flag `--extra-vars`, that will take a space separated list of parameters like this:
```sh
/usr/bin/ansible-playbook -i ~/simple_inventory.txt site.yml --private-key ~/ansible.key --extra-vars "my_software_version=2.0 some_other_var=meep"
```

##### Vaulting sensitive information
Some of the ansible roles may require a vault password, this is used to encrypt and decrypt sensitive information, for instance passwords, licences or sensitive configuration files.
If you find yourself needing this, create a vault password file (just a text file with a password in it, here called `~vault_password.txt`) and use these commands:
```sh
# Encrypt files using ansible-vault
ansible-vault encrypt --vault-password-file ~/vault_password.txt roles/example/files/super_secret.conf
# Similarly to decrypt the file again
ansible-vault decrypt --vault-password-file ~/vault_password.txt roles/example/files/super_secret.conf
# Run a playbook that has vaulted information in:
/usr/bin/ansible-playbook -i ~/simple_inventory.txt site.yml --private-key ~/ansible.key --vault-password-file ~/vault_password.txt
```
Files do not need to be decrypted before running `ansible-playbook` so long as you specify the `--vault-password-file`.
It's possible to have more than one vault password, there are docs about it.


### Explanation of the file structure and syntax <a name="structure"></a>
#### Top level <a name="top_level"></a>
`roles` directory, `site.yml`, and ansible.cfg.

- `playbooks` contains the same sort of information as `site.yml`, this could be grouped by use case, and roles for each piece of software or feature.
- `roles` contains the instructions for how to configure various components, playbooks can refer to one or multiple roles.
- `site.yml` top level playbook here. We can separate some of these into their own playbooks to make them easier to manage, and we import them here. Playbooks are run in the order they are specified in this file (same for imported playbooks).
- `ansible.cfg`sets out some global rules for Ansible to follow. You should run `ansible-playbook` from the directory with this in, otherwise it may act strangely and will likely not run the tasks.

#### Roles directory <a name="roles"></a>
Inside `roles` directory: `example`, an example role showing some of the things you can do with Ansible.

You can create subdirectories for roles to help group them.

The tasks we want Ansible to do for us are detailed in `tasks/main.yml`.
Some of the groups and software have another directory called `files`, where we keep some small files which need transferring to the VM - these tend to be small scripts or icons.

The syntax for tasks is as follows:

```yml
- name: This can be anything, but should say what this section is doing
  module_name:
    option1: value_of_option1
    option2: 9999
```

The `module_name` can be any of the modules Ansible supports, for instance `yum`, `copy`, or `ini_file` and the options for those are available in the Ansible documentation. For example, the one for yum: [Ansible docs for yum module](https://docs.ansible.com/ansible/latest/modules/yum_module.html "Ansible docs for yum module").

#### Examples <a name="examples"></a>

We can write a couple of lines in `main.yml` to get yum to install the latest version of "cowsay" (provided you have root access):
```yml
- name: Install cowsay
  yum:
    name: cowsay
    state: latest
```
Or copy a file from the `files` directory:
```yml
- name: Copy script.sh which needs to run when user logs in
  copy:
    src: script.sh
    dest: /tmp/
    mode: 0700
```

You can find many more of these by searching the internet for `ansible module` and then the thing you want to do, you'll probably end up at their documentation, like the one for the yum example above.

#### Linting <a name="linting"></a>

We can lint our Ansible repository using `ansible-lint`.
The included GitHub workflow runs a linter when a pull request is created, but you can do it manually with:
```sh
# Install prerequisites
pip3 install --upgrade pip
pip3 install -r requirements_lint.txt
# Run linter
ansible-lint
```

**Note:** I have pinned the `ansible-lint` version in `requirements_lint.txt` for the moment because of a bug listed here: https://github.com/ansible/ansible-lint/issues/3452
