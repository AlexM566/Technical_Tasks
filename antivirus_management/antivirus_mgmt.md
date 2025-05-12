# Antivirus for Windows

We are going to use Microsoft Defender for this and for the Deployment we can use Intune or SCCM where we should configure a profile with Defender and right after we can roll out in phases to workstations. Everything can be configured as well with powershell and managed with ansible or Ansible AWX. It depends of course...

# Antivirus for Linux
For any *UNIX host we can laverage ansible to push the antivirus whitch is Microsoft Defender ATP. In this way we can manage the antivirus software
Offcourse there are configuration files that can be managed and pulled from repos
Everything can integrated in CI/CD pipelines for a full cycle and automate deployments at a large scale

# Monitoring
We can monitor both instances in the Defender portal, which is a great adition.


# Sample code for the bove

**Deployment WINDOWS**
```yaml
---
- name: Deploy and Configure Microsoft Defender for Endpoint
  hosts: windows_servers
  gather_facts: yes
  vars:
    mde_onboarding_url: "https:company-test.com/defender-onboardin.ps1"
    tenant_name: "your-tenant-name" 
    download_path: "C:\\Temp"
    scan_time: "0200"  # 2:00 AM

  tasks:
    - name: Download Defender
      win_get_url: 
        url: "{{ mde_onboarding_url }}"
        dest: "{{ download_path }}\\onboard.ps1"
    
    - name: Run onboarding script
      win_shell: "& '{{ download_path }}\\onboard.ps1' -Quiet"
    
    - name: Verify service
      win_service: defender
      start_mode: auto
      state: started
```

**Deployment Linux**
```yaml
---
name: Deploy ATP Defender for Linux
- hosts: linux
  become: yes
  tasks:
    - name: Install MDE agent
      apt:
        name: mdatp
        update_cache: yes

    - name: Onboard to MDE
      command: mdatp onboarding --tenant-id "<ID>" --tenant-key "<KEY>"

    - name: Enable real-time protection
      command: mdatp --set real-time-protection --value enabled
```


## Monitoring
We are gonna use a splunk query to automate monitoring, Because the data after scannin arrives "somewhere" of course, otherwise there is no sense.

```splunk
index="ms_defender" sourcetype="WinEventLog:Security"
    | stats latest (HealthStatus) as status by host
    | stats count by Computer, health_status, EventCode, OperatingSystem // Adding OS to be able to filter between WIndows and *UNIX
    | where status != "Healthy" 
```

## Alerting as Code in SPLUNK for Defender
We can configure allerting in Splunk by code baed on searches ofcourse 
This cane done and managed with git and implemented in the splunk server:
# $SPLUNK_HOME/etc/apps/my_alerts/local/savedsearches.conf  

```conf
[Defender_Unhealthy_Hosts]
search = index="ms_defender" sourcetype="WinEventLog:Security"
    | stats latest(HealthStatus) as status by host
    | stats count by host, status, EventCode, OperatingSystem
    | where status!="Healthy"
cron_schedule = */5 * * * *
dispatch.earliest_time = -5m@m
dispatch.latest_time   = now
alert_type = number of events
alert_comparator = greater than
alert_threshold = 0
actions = email
action.email.to = ops@example.com
action.email.subject = Defender alert: unhealthy host on $result.host$
alert.severity = 2
```