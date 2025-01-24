# 📊 Proxmox Monitoring Stack - Ansible Playbook

This Ansible playbook automates the deployment of a monitoring stack using Prometheus and Grafana, specifically designed for monitoring:

🖥️ - Proxmox VE hosts  
🚀 - Virtual Machines (Linux for now)  
📦 - LXC Containers  
📈 - System metrics via Node Exporter

The stack provides comprehensive monitoring and visualization of:

🔍 - Host system metrics  
🎯 - Container resources  
⚡ - VM performance  
🏥 - Proxmox cluster health  
📊 - Custom metrics via Node Exporter

This project can serve as a reference implementation for:

🏠 - Homelab environments  
🌐 - Small to medium Proxmox deployments  
🛠️ - Self-hosted monitoring solutions  
📚 - Infrastructure automation examples

> **🚧 Note**: This is an actively developed project. Future updates will include:
>
> ✨ - Additional dashboard templates  
> 🚨 - Alert manager integration  
> 🔌 - More exporters for specialized monitoring  
> 📖 - Extended documentation and best practices

## ⚡ Prerequisites

🔧 - Ansible installed on the control machine  
💻 - Target server running Ubuntu (adjust if using different OS)  
🔑 - SSH access to the target server  
🍎 - For macOS users: Set `OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES` environment variable

## 📁 Directory Structure

```
monitoring-ansible/
├── inventory/
│   └── hosts.ini
├── group_vars/
│   └── all/
│       ├── vars.yml
│       └── vault.yml
├── roles/
│   ├── common/
│   ├── monitoring_base/
│   ├── prometheus/
│   ├── grafana/
│   ├── node_exporter/
├── templates/
│   └── monitoring/
│       ├── prometheus/
│       │   ├── docker-compose.yml.j2
│       │   └── prometheus.yml.j2
│       └── grafana/
│           ├── docker-compose.yml.j2
│           └── grafana.ini.j2
├── site.yml
└── ansible.cfg
```

## 🚀 Getting Started

### 1️⃣ Clone and Configure Inventory

1. Clone this repository
2. Create your inventory file from the example:
   ```bash
   cp inventory/hosts.ini.example inventory/hosts.ini
   ```
3. Edit `inventory/hosts.ini` with your actual server details:
   - Update IP addresses
   - Set correct SSH users
   - Add/remove hosts as needed

### 2️⃣ Configure Vault

1. Create your vault file from the example:
   ```bash
   cp group_vars/all/vault.yml.example group_vars/all/vault.yml
   ```
2. Create a secure vault password:
   ```bash
   openssl rand -base64 64 > .vault_pass
   chmod 600 .vault_pass
   ```
3. Edit the vault file with your secrets:
   ```bash
   ansible-vault edit group_vars/all/vault.yml
   ```
   ⚠️ Make sure to change all default passwords!

### 3️⃣ Verify Configuration

1. Test your setup:

   ```bash
   # List all hosts to verify inventory
   ansible-inventory --list

   # Test connection to hosts
   ansible all -m ping
   ```

## 🎮 Usage

1. Run the playbook:

   ```bash
   ansible-playbook site.yml
   ```

2. For testing specific hosts:
   ```bash
   ansible-playbook site.yml --limit monitoring_hosts
   ```

## 🌐 Access Services

🔍 - Prometheus: http://your-server-ip:9090
📊 - Grafana: http://your-server-ip:3000

⚠️ Default Grafana credentials are in `group_vars/all/vault.yml` - make sure to change these in production!

## 🔒 Security Notes

1. Never commit sensitive files:
   - 📁 `inventory/hosts.ini`
   - 🔑 `group_vars/all/vault.yml`
   - 🔐 `.vault_pass`
2. Always change default passwords before deploying to production
3. Use strong passwords for all services
4. Consider using different vault files for different environments

## 🔧 Adding New Secrets

1. Add new secrets to `group_vars/all/vault.yml`:

   ```bash
   ansible-vault edit group_vars/all/vault.yml
   ```

2. Reference them in `group_vars/all/vars.yml`:
   ```yaml
   my_variable: "{{ vault_my_variable }}"
   ```

## 🚨 Troubleshooting

If you encounter permission issues:

1. 🔑 Ensure your SSH key is properly set up
2. 👑 Check that your user has sudo privileges
3. 🔐 Verify the vault password is correct

### macOS Users

If you're running this playbook from macOS and encounter fork safety issues with Node Exporter installation:

```bash
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

Add this to your `~/.zshrc` or `~/.bash_profile` to make it permanent:

```bash
echo 'export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES' >> ~/.zshrc
```
