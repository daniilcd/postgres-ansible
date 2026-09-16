
PostgreSQL 18 Ansible Deployment

Automated deployment and configuration of PostgreSQL 18 on Ubuntu using Ansible.

The project provisions a PostgreSQL server from a clean Ubuntu installation, configures server settings and client authentication, and creates an application database and user.

## Features

- PostgreSQL 18 installation from the official PGDG repository
- Automated PostgreSQL configuration
- Custom `production.conf`
- Remote client access configuration
- `pg_hba.conf` managed through Ansible
- SCRAM-SHA-256 authentication
- Application database and role creation
- Secure password storage with Ansible Vault
- Idempotent Ansible deployment
- PostgreSQL service restart/reload handlers

## Architecture

```text
Ansible Control Node
       WSL / Ubuntu
            |
            | SSH
            v
    Ubuntu Server / VM
            |
            v
      PostgreSQL 18
            |
       +----+----+
       |         |
 production.conf pg_hba.conf
       |
       v
    app_db
       |
    app_user
```

## Project Structure

```text
postgres-ansible/
├── ansible.cfg
├── inventory/
│   ├── hosts.ini
│   └── group_vars/
│       └── postgres/
│           └── vault.yml
├── playbooks/
│   ├── postgres.yml
│   └── test.yml
└── roles/
    └── postgresql/
        ├── defaults/
        │   └── main.yml
        ├── handlers/
        │   └── main.yml
        ├── tasks/
        │   └── main.yml
        └── templates/
            ├── pg_hba.conf.j2
            └── production.conf.j2
```

## Requirements

### Control Node

- Ansible
- SSH client
- `community.postgresql` Ansible collection

Install the required collection:

```bash
ansible-galaxy collection install community.postgresql
```

### Managed Node

- Ubuntu
- SSH access
- Python

Additional PostgreSQL dependencies are installed automatically by the role.

## Inventory

Example:

```ini
[postgres]
postgres_server ansible_host=192.168.12.19 ansible_user=daniil
```

Replace the IP address and SSH user with values for your environment.

## Configuration

Main PostgreSQL variables are stored in:

```text
roles/postgresql/defaults/main.yml
```

Example:

```yaml
postgresql_version: "18"

postgresql_shared_buffers: "512MB"
postgresql_effective_cache_size: "1536MB"
postgresql_work_mem: "8MB"
postgresql_maintenance_work_mem: "128MB"

postgresql_max_connections: 50

postgresql_max_wal_size: "2GB"
postgresql_checkpoint_timeout: "15min"

postgresql_app_user: "app_user"
postgresql_app_db: "app_db"
```

## Secrets

The application database password is stored using Ansible Vault.

Create the encrypted Vault file:

```bash
ansible-vault create inventory/group_vars/postgres/vault.yml
```

Example variable inside the encrypted file:

```yaml
postgresql_app_password: "your_secure_password"
```

The encrypted Vault file can safely be stored in the repository, while the Vault password must remain private.

## Deployment

Check the playbook syntax:

```bash
ansible-playbook \
  -i inventory/hosts.ini \
  playbooks/postgres.yml \
  --syntax-check \
  --ask-vault-pass
```

Deploy PostgreSQL:

```bash
ansible-playbook \
  -i inventory/hosts.ini \
  playbooks/postgres.yml \
  -K \
  --ask-vault-pass
```

Ansible will configure the server only when changes are required.

## Remote PostgreSQL Access

PostgreSQL is configured to accept TCP connections from the allowed network.

Example connection:

```text
Host:     192.168.12.19
Port:     5432
Database: app_db
User:     app_user
```

Authentication is performed using SCRAM-SHA-256.

## Verification

Check PostgreSQL:

```bash
sudo -u postgres psql -c "SELECT version();"
```

Check listening interfaces:

```bash
sudo -u postgres psql -c "SHOW listen_addresses;"
```

Check the PostgreSQL service:

```bash
systemctl status postgresql
```

## Purpose

This project was created to practise PostgreSQL administration, Linux server configuration, Ansible automation, and Infrastructure as Code principles.

The goal is to make PostgreSQL deployment reproducible: a new Ubuntu server can be configured consistently by running the Ansible playbook instead of repeating the setup manually.
