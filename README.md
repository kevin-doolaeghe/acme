# ACME - Automated Certificate Management Environment

## Author

**Kevin DOOLAEGHE**

## Linux setup

There are several solutions :
* ACME.sh shell script running inside a Docker container;
* [Nginx Proxy Manager](https://nginxproxymanager.com/) with Docker;
* [Traefik Proxy](https://doc.traefik.io/traefik/) with Docker

## Windows setup

* Install `Posh-ACME` :
```
Install-Module -Name Posh-ACME -Scope AllUsers
Install-Module -Name Posh-ACME.Deploy -Scope AllUsers
```

* Enable Powershell script execution :
```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

* Generate certificate :
```
Invoke-Expression '$pArgs = @{
    DuckToken = (Read-Host -Prompt "Token" -AsSecureString)
    DuckDomain = (Read-Host -Prompt "Domain")
}
$certNames = "$($pArgs.DuckDomain).duckdns.org","*.$($pArgs.DuckDomain).duckdns.org"
$email = (Read-Host -Prompt "Email")
New-PACertificate $certNames -UseSerialValidation -Contact $email -Plugin DuckDNS -PluginArgs $pArgs -Verbose'
```
⚠️ Generated certificates are located in `%LOCALAPPDATA%\Posh-ACME` directory.

* Install generated certificates :
```
Install-PACertificate
```

* Schedule a task to auto-renew certificates :
```
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument '-Command "&{Submit-Renewal ; Install-PaCertificate}"'
$trigger = New-ScheduledTaskTrigger -Daily -At 5:00AM
$principal = New-ScheduledTaskPrincipal -UserID "NT AUTHORITY\SYSTEM" -LogonType ServiceAccount -RunLevel Highest
$settings = New-ScheduledTaskSettingsSet -MultipleInstances Parallel
Register-ScheduledTask -TaskName "Renew certificates" -Trigger $trigger -Action $action -Settings $settings -Principal $principal -Description "Renew certificates using Posh-ACME"
```

* Force certificate renewal and replacement for `Test` website in IIS :
```
Submit-Renewal -Force | Set-IISCertificate -SiteName 'Test' -RemoveOldCert -Verbose
```

## References

* [DNS providers working with Let's Encrypt DNS validation](https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438)
* [Certbot](https://certbot.eff.org/)
* [Certbot - Github repository](https://github.com/certbot/certbot)
* [Generating HTTPS Certificate using DNS a Challenge through Let's Encrypt](https://web.navan.dev/posts/2020-11-17-Lets-Encrypt-DuckDns.html)
* [ACME.sh shell script - Github repository](https://github.com/acmesh-official/acme.sh)
* [SNB Forums - DuckDNS with Let's Encrypt using ACME.sh](https://www.snbforums.com/threads/duckdns-with-letsencrypt.86114/)
* [Posh-ACME Powershell Module](https://poshac.me/)
* [Posh-ACME - DuckDNS plugin](https://poshac.me/docs/v4/Plugins/DuckDNS/)
* [Posh-ACME.Deploy](https://github.com/rmbolger/Posh-ACME.Deploy)
* [Posh-ACME - Github repository](https://github.com/rmbolger/Posh-ACME)
* [Win-ACME](https://www.win-acme.com/)
* [Win-ACME - Cloudflare plugin](https://www.win-acme.com/reference/plugins/validation/dns/cloudflare)
* [Youtube - How to Install Lets Encrypt Certificates on IIS with Autorenew](https://www.youtube.com/watch?v=vbk5kUT7GeY)
* [Youtube - How to Install Let’s Encrypt on Windows Server 2019](https://www.youtube.com/watch?v=XA5Hn9Ifnd4)
