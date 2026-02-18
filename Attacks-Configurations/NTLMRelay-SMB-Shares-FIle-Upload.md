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

## Attack

- Criacao do .lnk
<img width="1360" height="217" alt="ipblur" src="https://github.com/user-attachments/assets/1ec37377-4801-4f02-8f85-888ef5315d1b" />

- Iniciar Responder
  <img width="619" height="675" alt="r3" src="https://github.com/user-attachments/assets/f2e110f7-0504-40c5-98e9-43d93e08f412" />


- Click no arquivo/pasta
<img width="1158" height="662" alt="rel1" src="https://github.com/user-attachments/assets/498e868a-c14a-4e57-a980-0eabb0071aae" />

- Recebenmos NTLMv2
  <img width="921" height="447" alt="r4" src="https://github.com/user-attachments/assets/cf000a85-898b-42bd-af33-fb0226c8d9d6" />


## Prevencao
- Rede: Bloquear SMB na saída | Desabilitar Protocolos Legados LLMNR, NetBIOS-NS e mDNS em toda a rede via Política de Grupo
- Sistema: Desabilitar Autenticação NTLM


## Wazuh log (nao precisamos rule pois ja e detectado por Suricata)
<img width="1881" height="471" alt="wazuh-alert-com-suricata" src="https://github.com/user-attachments/assets/f8a083a2-d522-4486-94d0-fe3b1ea2be5a" />








  



