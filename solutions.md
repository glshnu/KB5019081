# Workarrond um Zeit zu gewinnen!
für alle, bei denen z.B keine Dienste mehr mit Domänenanmeldungen ausgeführt werden können, 
oder aber Laptops keine WLAN Verbindung über ein Zertifikat aufbauen können.

Diese 3 Registryeinträge in alle DCs einfügen:

> reg add "HKLM\SYSTEM\CurrentControlSet\services\kdc" /v KrbtgtFullPacSignature /t REG_DWORD /d 0 /f  
> reg add "HKLM\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters" /v RequireSeal /t REG_DWORD /d 0 /f  
> reg add "HKLM\SYSTEM\CurrentControlSet\services\kdc" /v ApplyDefaultDomainPolicy /t REG_DWORD /d 0 /f  

## laut Microsoft Support sollte dieser Eintrag ausreichen
> reg add "HKLM\SYSTEM\CurrentControlSet\services\kdc" /v ApplyDefaultDomainPolicy /t REG_DWORD /d 0 /f  

https://www.reddit.com/r/sysadmin/comments/ypbpju/patch_tuesday_megathread_20221108/  
https://twitter.com/fabian_bader/status/1590339101580222464**  
