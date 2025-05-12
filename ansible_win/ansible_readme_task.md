# Ansible Windows Service & Application  Management 


## Implementation

### ✅ Services Selection
- **Service with parameters**: Prometheus Node Exporter
  - Configurable parameters include collectors, listen address, port
  - Parameters are passed during installation using the `arguments` option
  
- **Service without complex parameters**: FileZilla
  - Simple installation without custom parameters
  - Basic service configuration

### ✅ Multi-Server Architecture
- Implementation includes proper inventory structure with:
  - Environment groups (production, development, testing)
  - Host grouping based on reachability
  - Group variables for configuration

### ✅ WinRM Configuration
  -  configured WinRM in inventory variables:
  - Using credssp transport for enhanced security
  - Credentials stored securely using Ansible Vault

### ✅ Conditional Execution
- Services are enabled/disabled via variables:
  - `pack_prometheus_node_exporter_enabled`
  - `pack_filezilla_enabled`

## WinRM's Role in Windows Automation

WinRM (Windows Remote Management) is essential for Ansible's Windows automation:
- Provides the communication protocol between Ansible control node and Windows hosts
- Enables remote PowerShell execution
- Allows secure credential handling using various authentication methods
- Configured in this project using credssp transport for enhanced security

## Ansible Playbook Usage for Service Management

### Checking Service Status
```yaml
- name: Check service status
  win_service:
    name: service_name
  register: service_result

- name: Display service status
  debug:
    msg: "Service {{ service_name }} is {{ service_result.state }}"
```

### Starting a Service
```yaml
- name: Start the service
  win_service:
    name: service_name
    state: started
```

### Stopping a Service
```yaml
- name: Stop the service
  win_service:
    name: service_name
    state: stopped
```

### Restarting a Service
```yaml
- name: Restart the service
  win_service:
    name: service_name
    state: restarted
```

## Key Ansible Modules for Windows Service Management

| Module | Purpose |
|--------|---------|
| `win_service` | Managing Windows services (start, stop, restart, configure) |
| `win_package` | Installing MSI/EXE applications |
| `win_get_url` | Downloading files from HTTP/HTTPS/FTP sources |
| `win_file` | Managing files and directories |
| `win_shell`/`win_command` | Running PowerShell/CMD commands |
| `win_feature` | Managing Windows features |
