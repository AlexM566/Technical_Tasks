# Automate the management of windows updates

There are some ways to Automate the windows updates both for workstaton and servers:
- Central Management: via Intune or SCCM  ( prefered way to do it)-> Intune
- Standalone Process with PowerShell scripting: with Powershell Windows Update module

There is also a recommended way of rollout:
- Pilot group ( 10-50 workstations)
- BETA pilot ( arrount 25-35 % of the hosts) 
- PROD deployment (Remaining)

# Capture Success Information

THis can be done with Event Viewe from source: WIndowsUpdateCLient


# Capture Failure Information
We can capture failure from the host where event id is is : Event ID 20
- collect the KB number, size, severity


## Retry logic:

### Collecting success/failure 
After each deployment run WindowsUpdate module result and log success and unssecc correlated with ResultCode, timestamp, hostname, eventID, HResults.
This can be centralized in database (non-relation I would sugest), with a SIEM tool or simple in CSV file.
If done with Intune this can be seen centralized in Intune Console.

#### Retry
- Read the log file, db etc.
- Group by host and retry each machine
- Invoke the script again
- Count retries in csv, db etc
- Apply manual intervention if needed.


# Sample code for Deployment, Monitoring, Alarming

## Deployment
We are doing this one with powershell this time. BUT Intune / SCCM WOULD be recommanded 

```powershell
param([switch]$AutoReboot)

# 1. Scan & install all applicable Win updates
$session  = New-Object -ComObject Microsoft.Update.Session
$searcher = $session.CreateUpdateSearcher()
$updates  = $searcher.Search("IsInstalled=0 and Type='Software'").Updates

$installer         = $session.CreateUpdateInstaller()
$installer.Updates = $updates
$result            = $installer.Install()

# 2. Log results (success or failure)
$record = [PSCustomObject]@{
  Timestamp    = Get-Date
  ComputerName = $env:COMPUTERNAME
  ResultCode   = $result.ResultCode
  HResults     = ($result.GetUpdateResults() | ForEach-Object HResult) -join ';'
}
$record | Export-Csv C:\Logs\WUResults.csv -NoTypeInformation -Append

# 3. Reboot if required
if ($AutoReboot -and $result.RebootRequired) {
    Restart-Computer -Force
}
```
## Monitoring and alerting
Let'see the monitoring and alerting

```powershell
# 1. Load any failures from the last 24 hours
$since    = (Get-Date).AddDays(-1)
$logPath  = "C:\Logs\WUResults.csv"

$failures = Import-Csv $logPath |
    Where-Object {
        [datetime]$_."Timestamp" -gt $since -and
        $_.ResultCode -ne '2'
    }

# 2. If any failures, send a summary email
if ($failures) {
    $body = $failures |
            Select-Object ComputerName,Timestamp,ResultCode,HResults |
            Out-String

    Send-MailMessage `
      -To "ops-team@company.com" `
      -Subject "ALERT: Windows Update Failures" `
      -Body $body `
      -SmtpServer "smtp.company-test.com"
}
```
# **IMPORTANT!**
**This is a simple way to do it with Powershell but I would recommand Intune for PROD in an Enterprise level!**