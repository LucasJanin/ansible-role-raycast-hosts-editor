# Ansible Role: Raycast Hosts Editor

This Ansible role creates VSCode (or compatible editor like Windsurf) Raycast scripts for remote hosts. It generates scripts that allow you to quickly open VSCode connected to remote hosts via SSH using Raycast on macOS.

## Features

- Generate a script for each host of the inventory.  
- It can be configured with a custom editor.  
- The path of the Raycast script can be configured.  

## Screenshot
![Raycast Hosts Editor](images/raycast-hosts-editor.png)

## Requirements

- Ansible 2.9 or higher
- Raycast installed on the target macOS system
- Visual Studio Code or compatible editor with Remote SSH extension installed
- SSH access configured between the target macOS system and the remote hosts

## Role Variables

```yaml
# Required variables
user: "username"                          # The username on the target macOS system
host: "hostname"                          # The hostname of the target macOS system
home: "/Users/username/"                  # The home directory on the target macOS system

# Optional variables with defaults
raycast_scripts_path: ".raycast/scripts"  # Path to Raycast scripts directory relative to home
editor_name: "VSCode"                     # Name of the editor to use in script titles
editor_path: "code"                       # Command to launch the editor
editor_alias: "vs"                        # Prefix for script filenames
```

## Dependencies

No dependencies on other roles.

## Example Playbook

```yaml
- name: Create Raycast scripts for remote hosts
  hosts: localhost
  gather_facts: false
  vars:
    management_user: "you"
    management_host: "your-mac"
    management_home: "/Users/you/"
  tasks:
    - name: Include raycast editor host role
      include_role:
        name: ansible-role-raycast-hosts-editor
      vars:
        user: "{{ management_user }}"
        host: "{{ management_host }}"
        home: "{{ management_home }}"
```

## Testing

The role includes a test playbook and inventory that can be used to verify functionality. To run the tests:

```bash
cd ansible-role-raycast-hosts-editor
ansible-playbook -i tests/inventory.ini tests/test.yml
```

The test creates Raycast scripts in a temporary directory (`~/tmp/raycast/scripts`) to avoid modifying your actual Raycast scripts during testing.

## Inventory Example

Here's an example inventory file format that works with this role:

```ini
[all]
ansible     ansible_host="192.168.1.10"   alias="a"     vspath="/home/ansible/ansible"
docker      ansible_host="192.168.1.11"   alias="d"     vspath="/data/docker"
homepage    ansible_host="192.168.1.12"   alias="hp"    vspath="/opt/homepage/config"

[all:vars]
ansible_user="ansible"
```

The role will automatically use the host variables from your inventory:

- `ansible_host`: The IP address or hostname of the remote host
- `ansible_user`: The SSH username to connect to the remote host
- `alias`: A short alias for the host (used in script names)
- `vspath`: The path to open in VSCode on the remote host

## New Method

Starting with version 2.0, this role now directly uses Ansible's host variables instead of requiring separate lists. This simplifies configuration and makes the role more maintainable.

### Template Changes

The template now directly accesses host variables:

```bash
#!/bin/bash
# Required parameters:
# @raycast.schemaVersion 1
# @raycast.title {{ editor_name }} {{ hostname }}
# @raycast.mode silent
#
# Optional parameters:
# @raycast.icon 📜
# @raycast.packageName Raycast Scripts
#
# Documentation:
# @raycast.description Open {{ editor_name }} on {{ hostname }}{% if hostvars[hostname].vspath %} at {{ hostvars[hostname].vspath }}{% endif %}
#
# @raycast.author Lucas Janin
# @raycast.authorURL https://github.com/LucasJanin

{{ editor_path }} --folder-uri "vscode-remote://ssh-remote+{{ hostvars[hostname].ansible_user | default('ansible') }}@{{ hostname }}{% if hostvars[hostname].vspath %}{{ hostvars[hostname].vspath }}{% endif %}"
```

### Task Changes

The task that creates the scripts has been simplified:

```yaml
- name: Create Raycast scripts locally
  ansible.builtin.template:
    src: "{{ role_path }}/templates/raycast-hosts-editor.j2"
    dest: "{{ temp_scripts_dir.path }}/{{ editor_alias }}-{{ item }}.sh"
    mode: '0755'
  loop: "{{ ansible_play_hosts_all }}"
  vars:
    hostname: "{{ item }}"
  delegate_to: localhost
```

This approach eliminates the need for pre-tasks that extract values from the inventory into separate lists.

## License

MIT

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin feature/my-new-feature`)
5. Create a new Pull Request

## Author Information

Lucas Janin
- Mastodon: [https://mastodon.social/@lucas3d](https://mastodon.social/@lucas3d)
- Website: [https://www.lucasjanin.com](https://www.lucasjanin.com)
- GitHub: [github.com/lucasjanin](https://github.com/lucasjanin)
- LinkedIn: [linkedin.com/in/lucasjanin](https://linkedin.com/in/lucasjanin)

