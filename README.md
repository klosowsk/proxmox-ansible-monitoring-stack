# 📊 Proxmox Monitoring Stack - Ansible Playbook

This Ansible playbook automates the deployment of a monitoring stack using Prometheus and Grafana, specifically designed for monitoring:

🖥️ - Proxmox VE hosts  
🚀 - Virtual Machines (Linux for now)  
📦 - LXC Containers  
📈 - System metrics via Node Exporter  
🔋 - UPS monitoring via NUT

The stack provides comprehensive monitoring and visualization of:

🔍 - Host system metrics  
🎯 - Container resources  
⚡ - VM performance  
🏥 - Proxmox cluster health  
📊 - Custom metrics via Node Exporter  
🔋 - UPS monitoring via NUT  
🚨 - Automated alerts via AlertManager

This project can serve as a reference implementation for:

🏠 - Homelab environments  
🌐 - Small to medium Proxmox deployments  
🛠️ - Self-hosted monitoring solutions  
📚 - Infrastructure automation examples

> **🚧 Note**: This is an actively developed project. Future updates will include:
>
> ✨ - Additional dashboard templates  
> 🔌 - More exporters for specialized monitoring  
> 📖 - Extended documentation and best practices

## ⚡ Prerequisites

🔧 - Ansible installed on the control machine  
💻 - Target server running Ubuntu (adjust if using different OS)  
🔑 - SSH access to the target server  
🍎 - For macOS users: Set `OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES` environment variable

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
🚨 - AlertManager: http://your-server-ip:9093 (when enabled)

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

## 🚨 Alerting System

The monitoring stack includes an optional alerting system using AlertManager with Slack integration.
The system monitors by default:

### Host Availability

- 🔴 **HostDown**: Critical alert when a host is unreachable for more than 1 minute
- ⚠️ **HostHighDowntime**: Warning when host uptime falls below 95% in 24 hours

### Resource Usage

- 💻 **HighCPUUsage**: Warning when CPU usage exceeds 85% for 5 minutes
- 🧠 **HighMemoryUsage**: Warning when memory usage exceeds 85% for 5 minutes
- 💾 **DiskSpaceRunningOut**: Warning when disk usage exceeds 85% for 5 minutes

### UPS Monitoring

The stack includes optional UPS monitoring via Network UPS Tools (NUT):

- 🔋 **UPSOnBattery**: Warning when UPS switches to battery power
- ⚡ **UPSLowBattery**: Critical alert when UPS battery level falls below 20%
- 📈 **UPSHighLoad**: Warning when UPS load exceeds 80% for 5 minutes

### Alert Configuration

- Critical alerts repeat every hour
- Warning alerts repeat every 12 hours
- All alerts include host instance and service information
- Alerts are automatically resolved when conditions return to normal

### Setting Up Alerts

1. Configure Slack integration:

   ```bash
   # Edit vault file
   ansible-vault edit group_vars/all/vault.yml

   # Add Slack configuration
   vault_slack_webhook_url: "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
   vault_slack_channel: "#alerts"
   ```

2. Alerts are automatically enabled when Slack webhook URL is configured
3. To disable alerts, remove or leave empty the `vault_slack_webhook_url`

### Setting Up NUT Monitoring

1. Enable NUT monitoring in your variables:

   ```yaml
   # group_vars/all/vars.yml
   nut_enabled: true
   ```

2. Set your NUT server variables:

   ```yaml
   # group_vars/all/vars.yml
   nut_exporter_port: 9199
   ```

3. Configure NUT connection details in vault:

   ```bash
   # Edit vault file
   ansible-vault edit group_vars/all/vault.yml

   # Add NUT configuration
   vault_nut_host: "192.168.1.12"
   vault_nut_port: 3493
   vault_nut_username: "monuser"      # NUT username
   vault_nut_password: "secret"       # NUT password
   ```

4. The NUT exporter will be automatically deployed when `nut_enabled` is true
5. Metrics will be available in Prometheus and Grafana
6. Default alerts will be configured for common UPS events

Common customizations for NUT alerts can be found in:

```
templates/monitoring/prometheus/rules/nut_alerts.yml.j2
```

You can adjust:

- Battery level thresholds (default: 20% for low battery)
- Load thresholds (default: 80% for high load)
- Alert timing and severity levels

## 🔧 Customizing Alerts

Alert thresholds can be adjusted in:

```
templates/monitoring/prometheus/rules/node_alerts.yml.j2
```

Common customizations:

- Adjust CPU/Memory thresholds (default: 85%)
- Change alert timing (default: 5m for resource alerts)
- Modify uptime requirements (default: 95% over 24h)
- Add new alert rules for specific metrics
