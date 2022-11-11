**[Links](links.md), [Lösungsansätze](solutions.md)**

!! Aktuell scheint es Probleme mit den Patches zu geben !!
Siehe [Lösungsansätze](solutions.md)

# November-Update bringt 3 Patches für Domain-Controller (CVE-2022-37966, CVE-2022-37967, CVE-2022-38023)

Das kumulative Update KB5019081 beseitigt drei Schwachstellen im Kerberos- und Netlogon-Protokoll. 
Die Patches können erhebliche Auswirkungen haben und erfordern daher eine Überprüfung oder Anpassung des Systems. 
Die Änderungen lassen sich teilweise noch aufschieben, werden aber im nächsten Jahr vollständig durchgesetzt.

Angreifer können die Schwachstellen CVE-2022-37966 und CVE-2022-37967 im Kerberos-Protokoll ausnutzen, um erhöhte 
Privilegien zu erlangen. Bei CVE-2022-38023 gilt Entsprechendes für das Netlogon-Protokoll.

Der Grund für CVE-2022-37966 ist der schwache RC4_HMAC_MD5-Algorithmus, der bis dato als Standard verwendet wurde, 
solange man nicht explizit über die Gruppenrichtlinien einen anderen zugewiesen hat.

![image](https://user-images.githubusercontent.com/7205231/201438805-0fa92651-42ff-45bf-a518-b36933fbd600.png)

Durch das aktuelle Update ändert Microsoft den Default für Session-Keys von User-Objekten, denen noch kein 
Standardverfahren zugewiesen wurde, auf AES. In Folge dessen können im Eventlog eines DCs unter System Einträge 
mit der ID 42 erscheinen. Die Meldung dazu lautet:

The Kerberos Key Distribution Center lacks strong keys for account: accountname. You must update the password of 
this account to prevent use of insecure cryptography. See https://go.microsoft.com/fwlink/?linkid=2210019 to learn more.

In diesem Fall empfiehlt der Hersteller, das Passwort für krbtgt zurückzusetzen. 
Zu diesem Zweck stellt er das Script [New-KrbtgtKeys.ps1](https://github.com/microsoft/New-KrbtgtKeys.ps1) zur Verfügung.

Um herauszufinden, welche Konten RC4 verwenden und somit für CVE-2022-37966 anfällig sind, schlägt Microsoft folgende 
Abfrage des AD vor:

> ((msDS-SupportedEncryptionTypes & 0x3F) != 0) && ((msDS-SupportedEncryptionTypes & 0x38) == 0)

Unklar bleibt dabei jedoch, in welchem Kontext man sie verwenden kann und welcher Logik sie folgt.

Wenn RC4 bei einem Konto aktiviert ist, dann wird im Attribut msDS-SupportedEncryptionTypes das dritte Bit 
im 32-Bit Integer gesetzt, das entspricht dem Wert 0x4. Dies kann man so abfragen:

> Get-ADuser -Property msDS-SupportedEncryptionTypes `
> -Filter 'msDS-SupportedEncryptionTypes -band 0x4' |
> select name, msDS-SupportedEncryptionTypes

Alternativ ginge das über einen LDAP-Filter:

> Get-ADUser -Property msDS-SupportedEncryptionTypes -LDAPFilter '(msDS-SupportedEncryptionTypes:1.2.840.113556.1.4.803:=4)'

Im ADSI-Editor könnte man diese LDAP-Abfrage hinterlegen:

> (&(!(objectclass=computer))(msDS-SupportedEncryptionTypes:1.2.840.113556.1.4.803:=4))

![image](https://user-images.githubusercontent.com/7205231/201439229-d2e8841a-07ac-4834-b2d2-1ba37c730be7.png)

## Änderung des Kerberos-Protokolls
Als Reaktion auf CVE-2022-37967 signiert Microsoft künftig den Kerberos PAC Buffer, damit Angreifer diesen nicht mehr 
manipulieren können, um höhere Rechte zu erlangen. Diese Änderung bleibt bei ungültigen Signaturen vorerst noch folgenlos.

Am 13. Dezember 2022 aktiviert Microsoft den Audit-Modus, der Tickets mit ungültiger Signierung über einen Eintrag im 
System-Log meldet (ID 43 und ID 44). Ab dem 11. Juli 2023 wechseln DCs dann in den Enforcement-Modus, in dem Signaturen 
valide sein müssen. Admins können dann noch bis zum 10. Oktober 2023 den Audit-Modus konfigurieren, danach ist dieser 
nicht mehr verfügbar.

Für diesen Zweck ist der Registry-Key KrbtgtFullPacSignature vom Typ DWORD unter
> HKLM:\System\currentcontrolset\services\kdc
vorgesehen. 

Ein Wert von   
2 steht für den Audit Mode,  
3 für den Enforcement-Modus,  
0 deaktiviert das Feature und  
1 signiert die PAC Buffer, verifiziert dies aber nicht.  
Letzteres ist aktuell die Vorgabe.  

## Änderungen des Netlogon-Protokolls
Microsoft reagiert auf CVE-2022-38023 mit der vollständigen Umstellung von RPC Signing auf RPC Sealing für Netlogon. 
Das November-Update erzwingt ab sofort RPC Sealing, vorerst aber nur im Kompatibilitätsmodus. Dabei müssen nur 
Windows-Geräte RPC Sealing verwenden.

Ab dem 11. April 2023 verlangt Microsoft dieses Verfahren auch von Geräten mit einem anderen Betriebssystem. 
Allerdings können Admins dann noch bis zum 11. Juli in den Kompatibilitätsmodus zurückwechseln, danach ist 
das nicht mehr möglich.

Dies lässt sich ebenfalls über einen Registry-Schlüssel steuern, nämlich RequireSeal unter
> HKLM:\SYSTEM\CurrentControlSet\Services\Netlogon\Parameters

Dabei steht der Wert 1 für Kompatibilität und  
2 für die Durchsetzung von RPC Sealing.  

