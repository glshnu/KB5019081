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
