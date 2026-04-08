# Lab 04 -- Network Automation with Ansible

> **NetworkThinkTank Labs** -- Hands-on networking labs for engineers, by engineers. Blog: [networkthinktank.blog](https://networkthinktank.blog)

---

## Blog Article Summary

This lab accompanies our blog series on **network automation using Ansible**. Ansible is one of the most popular open-source automation tools for network engineers because it is agentless, uses simple YAML-based playbooks, and supports a wide range of network platforms including Cisco IOS, NX-OS, Arista EOS, and Juniper JunOS.

In this lab, you will set up an Ansible control node, build an inventory of network devices, write playbooks to automate common network tasks (configuration deployment, backup, compliance checking), and troubleshoot common Ansible networking issues that you will encounter in real-world automation projects.

---

## Lab Objectives

By completing this lab, you will be able to:

1. Install and configure Ansible for network automation
2. Build a structured inventory file with groups, variables, and host-specific data
3. Use Ansible network modules (ios_config, ios_command, nxos_config, etc.)
4. Write playbooks for configuration backup, deployment, and compliance auditing
5. Use Jinja2 templates to generate device configurations dynamically
6. Implement roles and organize playbooks for scalable automation
7. Troubleshoot common Ansible connectivity and module issues

---

## Lab Topology

```n    +---------------------------------------------+
    |          Ansible Control Node                |
    |          (Ubuntu/CentOS VM)                  |
    |          IP: 192.168.1.100                   |
    |          Python 3.x + Ansible 2.14+          |
    +---------------------+------------------------+
                          |
                          | Management Network
                          | 192.168.1.0/24
          +---------------+---------------+
          |               |               |
    +-----+-----+  +-----+-----+  +------+----+
    |   R1       |  |   R2       |  |   SW1      |
    | Cisco IOS  |  | Cisco IOS  |  | Cisco NX-OS|
    | .1         |  | .2         |  | .3         |
    | (Router)   |  | (Router)   |  | (Switch)   |
    +-----+------+  +-----+------+  +------+-----+
          |               |               |
          +---------------+---------------+
                          |
                   Data Plane Network
                   10.0.0.0/24
```

> Full topology diagrams are available in the [`/diagrams`](./diagrams) folder.

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Platform** | GNS3, EVE-NG, or Cisco CML + Linux VM (Ubuntu 22.04 recommended) |
| **Device Images** | Cisco IOSv, NX-OSv, or equivalent SSH-capable images |
| **Ansible Version** | Ansible 2.14+ (ansible-core) with `ansible.netcommon` and `cisco.ios` collections |
| **Knowledge** | Basic Linux CLI, YAML syntax, Python fundamentals, CCNA-level networking |
| **Tools** | SSH client, text editor (VS Code recommended), Git |

---

## Step-by-Step Instructions

### Phase 1 -- Environment Setup

1. Deploy the topology using your preferred platform (see `/lab-files`).
2. Install Ansible on your control node: `pip install ansible ansible-pylibssh`.
3. Install required Ansible collections: `ansible-galaxy collection install cisco.ios cisco.nxos`.
4. Configure SSH access on all network devices (enable SSH, set username/password).
5. Verify SSH connectivity from the Ansible control node to all devices.

### Phase 2 -- Inventory and Variables

6. Create an Ansible inventory file (`inventory.yml`) with device groups: `[routers]`, `[switches]`, `[all:vars]`.
7. Define connection variables: `ansible_network_os`, `ansible_connection=network_cli`, `ansible_user`, `ansible_password`.
8. Create group_vars and host_vars directories for variable organization.
9. Test connectivity: `ansible all -m ping -i inventory.yml` (note: use `ios_ping` or `cli_command` for network devices).

### Phase 3 -- Basic Playbooks

10. Write a playbook to gather device facts: `ios_facts` / `nxos_facts`.
11. Write a playbook to backup running configurations to local files.
12. Write a playbook to deploy a standard configuration (NTP, DNS, SNMP, banner) using `ios_config`.
13. Write a playbook to collect `show` command outputs using `ios_command`.

### Phase 4 -- Advanced Automation

14. Create Jinja2 templates for dynamic configuration generation.
15. Use `template` module to render configs and push them with `ios_config`.
16. Implement an Ansible role for "base_config" with tasks, templates, defaults, and handlers.
17. Write a compliance-check playbook that validates device configurations against a baseline.

### Phase 5 -- Verification & Troubleshooting

18. Run playbooks with increased verbosity: `ansible-playbook -vvv`.
19. Use `--check` mode (dry run) before applying changes.
20. Implement idempotency checks -- run playbooks multiple times and verify no unnecessary changes.
21. Add error handling with `block/rescue/always` and `ignore_errors`.

---

## Real Configurations

Ansible files and device configurations are provided in the [`/configs`](./configs) folder:

| File | Description |
|---|---|
| `inventory.yml` | Ansible inventory with device groups and variables |
| `ansible.cfg` | Ansible configuration file with common settings |
| `backup-playbook.yml` | Playbook to backup running configurations |
| `deploy-base-config.yml` | Playbook to deploy standard base configuration |
| `compliance-check.yml` | Playbook to audit device configurations |
| `templates/base_config.j2` | Jinja2 template for base configuration |
| `R1-config.txt` | Router R1 reference configuration |
| `R2-config.txt` | Router R2 reference configuration |
| `SW1-config.txt` | Switch SW1 reference configuration |

---

## What Can Go Wrong -- Common Issues & Fixes

### 1. SSH Connection Timeout / "Unable to connect to port 22"
- **Cause:** SSH not enabled on the network device, or management IP unreachable.
- **Fix:** Verify SSH is configured on the device (`show ip ssh`). Check management network reachability. Ensure `ansible_connection=network_cli` is set.

### 2. Authentication Failed
- **Cause:** Wrong username/password, or privilege level mismatch.
- **Fix:** Verify credentials in inventory or vault. Ensure `ansible_become=yes` and `ansible_become_method=enable` if enable password is needed.

### 3. Module Not Found: "ansible.module_utils.connection"
- **Cause:** Missing Ansible network collections or wrong Ansible version.
- **Fix:** Install required collections: `ansible-galaxy collection install cisco.ios`. Verify Ansible version: `ansible --version`.

### 4. Playbook Runs but No Changes Applied
- **Cause:** Idempotency -- the configuration already matches the desired state.
- **Fix:** This is expected behavior. Use `-v` to see details. Check with `show running-config` on the device.

### 5. Jinja2 Template Rendering Errors
- **Cause:** Undefined variables, syntax errors in template, or wrong variable names.
- **Fix:** Test templates locally with `ansible -m template`. Verify variable names in `group_vars`/`host_vars` match template references.

### 6. "command timeout triggered" During Long Operations
- **Cause:** Default command timeout (30s) is too short for large config pushes or slow devices.
- **Fix:** Increase timeout in `ansible.cfg`: `[persistent_connection] command_timeout = 60`. Or set per-task: `vars: ansible_command_timeout: 60`.

---

## References

- [Ansible Network Automation Documentation](https://docs.ansible.com/ansible/latest/network/index.html)
- [Ansible cisco.ios Collection](https://galaxy.ansible.com/cisco/ios)
- [Ansible cisco.nxos Collection](https://galaxy.ansible.com/cisco/nxos)
- [Jinja2 Template Designer Documentation](https://jinja.palletsprojects.com/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/tips_tricks/ansible_tips_tricks.html)
- [NetworkThinkTank Blog](https://networkthinktank.blog)

---

## Repository Structure

```nlab-04-network-automation-ansible/
|-- README.md
|-- configs/
|   +-- .gitkeep
|-- diagrams/
|   +-- .gitkeep
+-- lab-files/
    +-- .gitkeep
```

---

> **Pro Tip:** Always use `--check` mode first to preview changes before applying them to production devices. Combine with `--diff` to see exactly what lines will be added or removed.

**Happy Labbing!**