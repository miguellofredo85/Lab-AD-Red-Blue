### Recurso compartilhado para exploracao de subidas de arquivos .lnk o .scf

- criar uma pasta Shares no C:\ e fazer click direito
  <img width="1270" height="826" alt="smb1" src="https://github.com/user-attachments/assets/0e48ebd1-8cd9-490c-8b7b-0e4ff22f2581" />

- click no Share
  <img width="1083" height="745" alt="smb2" src="https://github.com/user-attachments/assets/e2c4c307-9e63-4692-be2e-61446ea612b9" />

- no Security → edit → Add → Everyone permisos full o read/write e aplicar todo
  <img width="1306" height="792" alt="smb3" src="https://github.com/user-attachments/assets/d0f35df1-a2c2-4b55-979c-12a7a391f421" />


- como guest
  PS C:\Windows\system32> net user guest /active:yes
The command completed successfully.

PS C:\Windows\system32> net user guest /passwordreq:no
The command completed successfully.

PS C:\Windows\system32> net user guest ""
User name                    Guest
Full Name

- Adicionar a Guest no Acces this computer from the network e tirarlo do Deny access…
<img width="1262" height="771" alt="smb4" src="https://github.com/user-attachments/assets/69d8f73c-7d9d-426e-b953-15c0ba9928f4" />


### Adding Schedule Task to Trigger LNK File on SMB Shares

- Salvar esse script no C:\Users\h.harper\lnk-trigger.ps1
while ($true) {
    $folder = "C:\Shares"
    $latest = Get-ChildItem -Path $folder -Filter "*.lnk" | Sort-Object LastWriteTime -Descending | Select-Object -First 1

    if ($latest) {
        Start-Process $latest.FullName
    }

    Start-Sleep -Seconds 60
    Remove-Item -Path "C:\Shares\*" -Recurse -Force
}




