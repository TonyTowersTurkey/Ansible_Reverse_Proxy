# Ansible Reverse Proxy

This repository contains an Ansible playbook to configure an OCI Ubuntu reverse proxy server with:

- system updates and upgrades
- Docker + Docker Compose
- Tailscale installation
- UFW firewall rules
- fail2ban
- Nginx Proxy Manager deployed with Docker

## Project structure

- `ansible.cfg` - Ansible configuration for inventory and roles.
- `playbook.yml` - Main playbook applying the `reverse-proxy` role.
- `inventory.ini` - Example inventory for a single reverse proxy host.
- `inventories/production/hosts` - Production inventory sample.
- `inventories/production/group_vars/all.yml` - Shared variables for the proxy host.
- `roles/reverse-proxy/` - Role that provisions the reverse proxy stack.

## Features

- Runs `apt update` and `apt upgrade -y`
- Installs Docker and Docker Compose plugin
- Enables Docker and adds the `ubuntu` user to the Docker group
- Creates an optional secondary sudo user with SSH key support
- Installs and enables Tailscale
- Configures UFW to allow only ports `22`, `80`, and `443`
- Installs and starts `fail2ban`
- Deploys Nginx Proxy Manager on ports `80`, `81`, and `443`

## Usage

1. Update `inventories/production/hosts` or `inventory.ini` with your cloud public IP.
2. Set your desired values in `inventories/production/group_vars/all.yml`.
3. Ensure the machine running Ansible has SSH access to the remote host.
   - Use `ansible_ssh_private_key_file` in inventory, or pass `--private-key` on the CLI.
   - Do not store private keys in the repo; keep them local to the machine you run from.
4. Run the playbook:

```bash
ansible-playbook playbook.yml -i inventory.ini
```

If you want to use the existing inventory path configured in `ansible.cfg`, run:

```bash
ansible-playbook playbook.yml
```

## Portable infrastructure

This repository is designed so any computer with Ansible and SSH access can redeploy or rebuild the reverse proxy.

- The inventory file declares the target host and SSH connection settings.
- Host-specific secrets such as private keys are kept out of source control.
- The playbook is idempotent, so re-running it should converge the target into the desired state.

## Variables

The role reads defaults from `roles/reverse-proxy/defaults/main.yml` and host-specific overrides from `inventories/production/group_vars/all.yml`.

Important variables:

- `reverse_proxy_create_admin_user` - whether to create a secondary sudo user
- `reverse_proxy_admin_user` - secondary admin username
- `reverse_proxy_admin_authorized_key` - public SSH key for the secondary user
- `reverse_proxy_tailscale_enabled` - whether to install Tailscale
- `reverse_proxy_tailscale_auth_key` - optional Tailscale auth key for unattended activation
- `reverse_proxy_npm_path` - install path for Nginx Proxy Manager
- `reverse_proxy_ufw_allow_ports` - open ports through `ufw`

## Notes

- This playbook is built for Ubuntu-based cloud instances.
- `reverse_proxy_tailscale_auth_key` can be left empty if you prefer to activate Tailscale manually after deployment.
- Nginx Proxy Manager admin UI is available on port `81` by default.
- OCI security lists / NSGs must still allow `22`, `80`, and `443` for external access.
