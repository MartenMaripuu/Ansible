# Ansible
# Ansible Fleet Backup Lab

This project automates daily system configuration (`/etc`) backups across multiple target virtual machines using Ansible.

## Setup Overview

* **Control Node (`ubuntu4`):** `172.18.25.30`
* **Target Fleet:** 
  * `vm2` (`172.18.25.35`)
  * `vm3` (`172.18.25.38`)
* **User:** `marten`

---

## Configuration Files

### Inventory (`hosts.ini`)
```ini
[fleet]
vm2 ansible_host=172.18.25.35 ansible_user=marten
vm3 ansible_host=172.18.25.38 ansible_user=marten

Playbook
---
- name: Daily Backup Playbook for Fleet VMs
  hosts: fleet
  become: yes
  vars:
    backup_dir: "/var/backups/fleet_backup"
    source_dir: "/etc"

  tasks:
    - name: Ensure backup directory exists
      ansible.builtin.file:
        path: "{{ backup_dir }}"
        state: directory
        mode: '0755'

    - name: Create timestamped archive of the target directory
      ansible.builtin.archive:
        path: "{{ source_dir }}"
        dest: "{{ backup_dir }}/backup-{{ ansible_date_time.date }}.tar.gz"
        format: gz
TESTING
ssh-copy-id marten@172.18.25.35
ssh-copy-id marten@172.18.25.38
echo "marten ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/marten
ansible fleet -i hosts.ini -m ping
ansible-playbook -i hosts.ini backup.yml
Final verification:ls -la /var/backups/fleet_backup
# Output confirmed: backup-2026-11-10.tar.gz
