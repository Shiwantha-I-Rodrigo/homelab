**~ *Ansible* Justice ~** <sub><sup>by Ann Leckie</sup></sub>

---

# ANSIBLE

## SUMMARY

Ansible is an open-source automation tool :
- Configuration management (setting up servers, installing software, managing files)
- Application deployment (rolling out code or updates across many systems)
- Task automation (running repetitive IT jobs)
- Orchestration (coordinating multiple machines or services to work together)

Ansible works in a “push” model (connects to servers (managed nodes) from a computer (control node) and pushes the desired configurations).
- SSH (Secure Shell) to connect to remote machines.
- YAML files (human-readable text files) to describe what needs to be done.

| Concept | Description |
| - | - |
| **Control Node** | The machine where Ansible is installed and commands are run.|
| **Managed Node** | The devices that Ansible controls.|
| **Inventory**    | A list (file) of managed nodes and their connection details.|
| **Modules**      | Prebuilt scripts that perform tasks (ie. install packages, copy files).|
| **Playbook**     | A YAML file defining *what* to do and *where*.|
| **Task**         | A single action within a playbook (e.g., “install nginx”).|
| **Role**         | A structured way to organize playbooks and tasks for reusability.|

Ansible can automate configuration, monitoring, and management of Cisco network devices.

For Cisco, Ansible uses network_cli or httpapi instead of the default Linux SSH.

```
/etc/ansible/hosts

[routers]
R1 ansible_host=192.168.1.10 ansible_network_os=ios ansible_connection=network_cli ansible_user=admin ansible_password=cisco
R2 ansible_host=192.168.1.11 ansible_network_os=ios ansible_connection=network_cli ansible_user=admin ansible_password=cisco

[switches]
SW1 ansible_host=192.168.1.20 ansible_network_os=ios ansible_connection=network_cli ansible_user=admin ansible_password=cisco
```

| Module | Purpose |
| - | - |
| `ios_config`   | Push configuration lines or templates to IOS devices.|
| `ios_facts`    | Gather device facts (hostname, interfaces, version, etc.).|
| `ios_command`  | Run show commands (ie. “show ip interface brief”).|
| `nxos_config`  | Manage NX-OS configurations.|
| `iosxr_config` | Manage IOS-XR devices.|
| `asa_config`   | Manage Cisco ASA firewall configs.|

---

## CONFIGUARATION

```
cisco_config.yml

- name: Configure Cisco Router
  hosts: routers
  gather_facts: no
  connection: network_cli

  tasks:
    - name: Configure hostname and interface
      ios_config:
        lines:
          - hostname R1
          - interface GigabitEthernet0/0
          - description Uplink_to_ISP
          - ip address 192.168.10.1 255.255.255.0
          - no shutdown
```

`ansible-playbook cisco_config.yml`

---

**ADVANCED USES**
- Dynamic configuration with Jinja2 templates (generate configs per site/device)
- Network state validation (ensure VLANs, routes, or ACLs match expectations)
- Backups (use ios_command to save “show running-config”)
- Firmware management (upload and verify images)
- Integration with CI/CD pipelines for network changes

**JINJA2**

1 - Dynamically generates configuration using Jinja2 templates.
2 - Pushes the configuration to routers.
3 - Validates the network state to ensure the configuration applied correctly.

```
# Files

cisco-automation/
├─ inventory.ini
├─ vars.yml
├─ router_config.j2
└─ deploy_and_validate.yml
```

inventory.ini --->
```
[routers]
R1 ansible_host=192.168.1.10 ansible_network_os=ios ansible_connection=network_cli ansible_user=admin ansible_password=cisco
R2 ansible_host=192.168.1.11 ansible_network_os=ios ansible_connection=network_cli ansible_user=admin ansible_password=cisco

```

vars.yml --->
```
routers:
  - hostname: R1
    loopback: 1.1.1.1
    interface: GigabitEthernet0/0
    ip: 10.0.0.1
    vlans: [10, 20]
  - hostname: R2
    loopback: 2.2.2.2
    interface: GigabitEthernet0/0
    ip: 10.0.0.2
    vlans: [10, 20]
```

router_config.j2 --->
```
hostname {{ item.hostname }}

interface {{ item.interface }}
 ip address {{ item.ip }} 255.255.255.0
 no shutdown
!

interface Loopback0
 ip address {{ item.loopback }} 255.255.255.255
!

{% for vlan in item.vlans %}
vlan {{ vlan }}
 name VLAN{{ vlan }}
{% endfor %}
```

deploy_and_validate.yml --->
```
---
- name: Deploy and validate Cisco routers
  hosts: routers
  gather_facts: no
  connection: network_cli

  vars_files:
    - vars.yml

  tasks:
    - name: Push dynamic configuration
      ios_config:
        lines: "{{ lookup('template', 'router_config.j2') }}"
      loop: "{{ routers }}"
      loop_control:
        loop_var: item

    - name: Validate VLANs
      ios_command:
        commands:
          - show vlan brief
      register: vlan_output

    - name: Assert required VLANs exist
      assert:
        that:
          - "'10' in vlan_output.stdout[0]"
          - "'20' in vlan_output.stdout[0]"
        fail_msg: "VLAN 10 or 20 is missing!"
        success_msg: "All required VLANs are present"

    - name: Validate interface status
      ios_command:
        commands:
          - show ip interface brief
      register: intf_output

    - name: Assert interface {{ item.interface }} is up
      assert:
        that:
          - "'up' in intf_output.stdout[0].split(item.interface)[1]"
        fail_msg: "Interface {{ item.interface }} is down!"
        success_msg: "Interface {{ item.interface }} is up"
      loop: "{{ routers }}"
      loop_control:
        loop_var: item
```

RUN ---> `ansible-playbook -i inventory.ini deploy_and_validate.yml`

---

## MONITORING & MAINTENANCE

| **Category** | **Command** | **Description** |
| - | - | - |
| **Installation & Version**     | `ansible --version`                                                             | Check Ansible version, Python environment, default inventory, and config file.|
| **Configuration Check**        | `ansible-config list` <br> `ansible-config dump --only-changed`                 | Show all configuration options or only changed overrides; troubleshoot Ansible behavior.|
| **Connectivity Test**          | `ansible all -i inventory.ini -m ping`                                          | Verify the control node can reach all managed hosts.|
| **Dependencies**               | `ansible-galaxy collection list` <br> `ansible-galaxy role list` <br> `pip list \| grep ansible`| Ensure required modules and collections (ie. `cisco.ios`) are installed.|
| **Upgrade Ansible**            | `pip install --upgrade ansible`                                                 | Keep the Ansible control node updated.|
| **Update Collections / Roles** | `ansible-galaxy collection install cisco.ios -f`                                | Ensure network automation modules are current.|
| **Clear Cache / Facts**        | Delete `~/.ansible/facts.d` or temporary files                                  | Avoid stale facts affecting playbook runs.|
| **Test Playbooks (Dry Run)**   | `ansible-playbook --check playbook.yml`                                         | Validate playbooks without making changes.|
| **Verbose Logging**            | `ansible-playbook -vvv playbook.yml`                                            | Debug and troubleshoot detailed task execution.|
| **Log File Maintenance**       | Configure `ansible.cfg`: <br> `[defaults] log_path = /var/log/ansible.log`      | Keep a permanent record of playbook runs for auditing.|
