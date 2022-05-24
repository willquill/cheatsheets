# Powershell

## Disable Windows Defender

[Source](http://wmug.co.uk/wmug/b/pwin/archive/2015/05/12/quickly-disable-windows-defender-on-windows-10-using-powershell)

In admin powershell:

`Set-MpPreference -DisableRealtimeMonitoring $true`

## Windows Server

### Allow management on server

`Enable-PSRemoting -SkipNetworkProfileCheck –Force`

To manage from client, run the following on client

Add to hosts file:

`10.1.20.11 dc1`

Where you have IP of server, hostname of server.

[Source](https://4sysops.com/archives/manage-windows-server-2019-with-admin-center-powershell-core-and-sconfig/)

All on client:

Start WinRM service

`Start-Service WinRM`

Enable the local account token filter policy

`Set-ItemProperty –Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System –Name LocalAccountTokenFilterPolicy –Value 1 -Type DWord`

Now you can add all computers to your TrustedHosts list

`Set-Item WSMan:\localhost\Client\TrustedHosts –Value *`

Download the latest x64 PowerShell Core release from Microsoft's GitHub releases page. Next, let's define a persistent PowerShell remoting session with the Server Core box (substituting your own server's IPv4 address, naturally) and copy the .msi to the root of its drive C:

`$session = New-PSSession -ComputerName dc1.paw.blue -Credential (Get-Credential)`

`Copy-Item C:\PowerShell-6.2.2-win-x64.msi C:\ -ToSession $session`

I had lots of issues getting the above to work.
[This](https://serverfault.com/questions/923799/cannot-remote-connect-to-windows-2019-core) fixed it.

Locally:

`winrm s winrm/config/client '@{TrustedHosts="dc1.paw.blue"}'`

On server:

`enable-psremoting -force`

`Set-PSSessionConfiguration -ShowSecurityDescriptorUI -Name Microsoft.PowerShell -Force`

A window will pop up - add your domain user and give Allow on everything

`enter-pssession -ConnectionUri http://10.1.20.11:5985 -credential paw\Administrator`

Find interface index number

`Get-NetAdapter`

Set IP address

`New-NetIPaddress -InterfaceIndex 3 -IPAddress 10.1.20.11 -PrefixLength 24 -DefaultGateway 10.1.20.1`

Set DNS to existing domain controller first if joining domain

`Get-DNSClientServerAddress –InterfaceIndex 2`

`Set-DNSClientServerAddress –InterfaceIndex 2 -ServerAddresses ("10.1.20.12","10.1.20.11")`

### Install AD Domain Services

`install-windowsfeature AD-Domain-Services`

### Create a new AD forest

#### New way for 2019

*Note: forestmode 7 is 2016. 2019 doesn't actually have its own mode. Same with domainmode.*

```powershell
Install-ADDSForest 
  -DomainName "paw.blue" 
  -CreateDnsDelegation:$false 
  -DatabasePath "C:\Windows\NTDS" 
  -DomainMode "7" 
  -DomainNetbiosName "PAW" 
  -ForestMode "7"
  -InstallDns:$true
  -LogPath "C:\Windows\NTDS"
  -NoRebootOnCompletion:$True
  -SysvolPath "C:\Windows\SYSVOL"
  -Force:$true
```

### Join an existing domain

`Add-Computer -Credential $(get-credential) -DomainName paw.blue -NewName ad2;Restart-Computer -force`

Check if joined domain:

`(Get-WmiObject -Class Win32_ComputerSystem).PartOfDomain`

Add this server as a domain controller

`Install-ADDSDomainController -DomainName "paw.blue" -credential $(get-credential)`

Allow Server Manager remotely

`Set-Item wsman:\localhost\Client\TrustedHosts Andromeda -Concatenate -Force`

`Set-Item wsman:\localhost\Client\TrustedHosts dc1.paw.blue -Concatenate -Force`

`$session = New-PSSession -ComputerName dc1.paw.blue -Credential (Get-Credential)`

`Invoke-Command $session -Scriptblock { Import-Module ActiveDirectory }`

`Import-PSSession -Session $session -module ActiveDirectory`

## Troubleshooting

`winrm set winrm/config/client @{TrustedHosts="10.1.20.40"}`

`dcpromo.exe /unattend /NewDomain:paw /ReplicaOrNewDomain:paw /NewDomainDNSName:paw.blue /DomainLevel:4 /ForestLevel:4 /SafeModeAdminPassword:<whateveryourpasswordis>`

`dcpromo.exe /unattend /NewDomain:paw /ReplicaOrNewDomain:paw /NewDomainDNSName:paw.blue /SafeModeAdminPassword:<whateveryourpasswordis>`

`dcpromo.exe /unattend /IgnoreLastDCInDomainMismatch`

## Certificates

`certreq -Submit -Attrib "CertificateTemplate:WebServer" -Config - panfw1.homelab.city.csr`

## Notes from provisioning a domain controller

[Source](https://sysadminguides.org/2017/04/12/server-build-script/)

```powershell
Clear
#Removes any previous password file
Remove-Item -Path C:\mysecurestring.txt -ErrorAction SilentlyContinue`

#Inputs for creds/domain
Do {
$Cname = Read-Host "Enter the intended name for the Server " #Ask for source Hyper-V server with current hosts
$Domain = Read-Host "Enter your domain <contoso.com>  "
$user = Read-Host "Enter your admin account username <admin01> "
$username = "$Domain\$user"
Read-host "Enter your password (It will be stored in an encrypted file C:\mysecurestring.txt)"  -assecurestring | convertfrom-securestring | out-file C:\mysecurestring.txt
Write-host " "
Write-host "Your inputs: $Cname | $Domain | $user" 
$answer = Read-host "Are you happy with your inputs <yes/no>? " 
    
    If ($answer -eq "yes")
    {
        $success = 1 }
    Elseif ($answer -eq "no")
    {
        Clear
        Write-host "--Cleared for re-input--"
        $success = 0}
   }
    While ($success -eq 0) 

#Inputs for adapter
Do {
Write-host "-Server adapter(s) info-"
Get-NetAdapter
Write-host " "  
$Adapter = Read-Host "Enter the intended adapter  "
$IP = Read-Host "Enter intended IP for the server "
$Subnet = Read-Host "Enter subnet mast (prefix length) <24> "
$DefaultG = Read-Host "Enter the default gateway "
$DNS = Read-Host "Enter your primary DNS "
$DNS2 = Read-Host "Enter your secondary DNS "
Write-host " "
Write-host "Your inputs: $Adapter | $IP | $Subnet | $DefaultG | $DNS | $DNS2"
$answer = Read-host "Are you happy with your inputs <yes/no>? "

    If ($answer -eq "yes")
    {
        $success = 2 }
    Elseif ($answer -eq "no")
    {
        Clear
        Write-host "--Cleared for re-input--"
        Write-host "Your inputs: $Cname | $Domain | $user" 
        $success = 1}
    } 
    While ($success -eq 1) 

Write-host " "
Write-host "Your current inputs: "
Write-host "$Cname | $Domain | $user" 
Write-host "$Adapter | $IP | $Subnet | $DefaultG | $DNS | $DNS2"

$OldCName = $env:COMPUTERNAME
$password = cat C:\mysecurestring.txt | convertto-securestring
$cred = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $password

#Configuring Network adapter
Write-Host " "
Write-Host "-Configuring $Adapter settings-"
New-NetIPAddress -InterfaceAlias $Adapter -IPAddress $IP -PrefixLength $Subnet -DefaultGateway $DefaultG
Set-DnsClientServerAddress -InterfaceIndex 3 -ServerAddresses $DNS,$DNS2
#Set-DnsClientServerAddress -InterfaceIndex 4 -ServerAddresses 127.0.0.1,10.1.20.1

Write-host " "
Write-host "-Adapter Settings Completed-"
Write-host " "

Write-host " "
$remote = Read-Host "Do you want to setup Remote Desktop <yes|no> "

if($remote -eq "yes"){

#Enable remote desktop
set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
netsh advfirewall firewall set rule name="Remote Desktop - User Mode (TCP-In)" new enable =Yes profile="domain,private"
netsh advfirewall firewall set rule name="Remote Desktop - User Mode (UDP-In)" new enable =Yes profile="domain,private"
set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
Write-Host "-Remote desktop setup-"
}
Write-host " "
$FileS = Read-Host "Do you want to join install the feature 'File Server' Enabling access to files on the server over the network? "
if($FileS -eq "yes"){
Install-WindowsFeature -name FS-FileServer
} 

Write-host " "
$domain = Read-Host "Do you want to join the server to the domain? "

if($domain -eq "yes"){

#Join domain/Rename
Rename-Computer -NewName $Cname
sleep 5
Add-Computer -ComputerName $OldCName -DomainName $Domain -credential $cred -force -Options JoinWithNewName,AccountCreate -restart
} 

#Removes password file
Remove-Item -Path C:\mysecurestring.txt -ErrorAction SilentlyContinue
```

## Automate WSUS on Windows 2016 Server Core

[Source](https://xenappblog.com/2017/automate-wsus-on-windows-2016-server-core/)

```powershell
Install-WindowsFeature -Name UpdateServices -IncludeManagementTools
New-Item -Path C: -Name WSUS -ItemType Directory
CD "C:\Program Files\Update Services\Tools"
.\wsusutil.exe postinstall CONTENT_DIR=C:\WSUS
 
Write-Verbose "Get WSUS Server Object" -Verbose
$wsus = Get-WSUSServer
 
Write-Verbose "Connect to WSUS server configuration" -Verbose
$wsusConfig = $wsus.GetConfiguration()
 
Write-Verbose "Set to download updates from Microsoft Updates" -Verbose
Set-WsusServerSynchronization -SyncFromMU
 
Write-Verbose "Set Update Languages to English and save configuration settings" -Verbose
$wsusConfig.AllUpdateLanguagesEnabled = $false           
$wsusConfig.SetEnabledUpdateLanguages("en")           
$wsusConfig.Save()
 
Write-Verbose "Get WSUS Subscription and perform initial synchronization to get latest categories" -Verbose
$subscription = $wsus.GetSubscription()
$subscription.StartSynchronizationForCategoryOnly()
 
 While ($subscription.GetSynchronizationStatus() -ne 'NotProcessing') {
 Write-Host "." -NoNewline
 Start-Sleep -Seconds 5
 }
 
Write-Verbose "Sync is Done" -Verbose
 
Write-Verbose "Disable Products" -Verbose
Get-WsusServer | Get-WsusProduct | Where-Object -FilterScript { $_.product.title -match "Office" } | Set-WsusProduct -Disable
Get-WsusServer | Get-WsusProduct | Where-Object -FilterScript { $_.product.title -match "Windows" } | Set-WsusProduct -Disable
 
Write-Verbose "Enable Products" -Verbose
Get-WsusServer | Get-WsusProduct | Where-Object -FilterScript { $_.product.title -match "Windows Server 2016" } | Set-WsusProduct
 
Write-Verbose "Disable Language Packs" -Verbose
Get-WsusServer | Get-WsusProduct | Where-Object -FilterScript { $_.product.title -match "Language Packs" } | Set-WsusProduct -Disable
 
Write-Verbose "Configure the Classifications" -Verbose
 
 Get-WsusClassification | Where-Object {
 $_.Classification.Title -in (
 'Critical Updates',
 'Definition Updates',
 'Feature Packs',
 'Security Updates',
 'Service Packs',
 'Update Rollups',
 'Updates')
 } | Set-WsusClassification
 
Write-Verbose "Configure Synchronizations" -Verbose
$subscription.SynchronizeAutomatically=$true
 
Write-Verbose "Set synchronization scheduled for midnight each night" -Verbose
$subscription.SynchronizeAutomaticallyTimeOfDay= (New-TimeSpan -Hours 0)
$subscription.NumberOfSynchronizationsPerDay=1
$subscription.Save()
 
Write-Verbose "Kick Off Synchronization" -Verbose
$subscription.StartSynchronization()
 
Write-Verbose "Monitor Progress of Synchronisation" -Verbose
 
Start-Sleep -Seconds 60 # Wait for sync to start before monitoring
 while ($subscription.GetSynchronizationProgress().ProcessedItems -ne $subscription.GetSynchronizationProgress().TotalItems) {
 #$subscription.GetSynchronizationProgress().ProcessedItems * 100/($subscription.GetSynchronizationProgress().TotalItems)
 Start-Sleep -Seconds 5
 }
```
