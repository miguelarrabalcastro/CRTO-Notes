```powershell
$LangList = Get-WinUserLanguageList
$LangList.Add("es-ES")
Set-WinUserLanguageList $LangList -Force
```
> Vamos a language settings(escribes "**la**" en windows) y seleccionas teclado en español.
