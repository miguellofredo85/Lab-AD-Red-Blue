👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação

Quando a Microsoft o lançou com o Windows Server 2008, o Group Policy Preferences (GPP) introduziu a capacidade de armazenar e usar credenciais em vários cenários, todos armazenados pelo AD no diretório de políticas no SYSVOL.

A chave de criptografia que o AD usa para criptografar os arquivos de política XML (a mesma para todos os ambientes do Active Directory) foi divulgada no Microsoft Docs, permitindo que qualquer pessoa descriptografe as credenciais armazenadas nos arquivos de política. Qualquer pessoa pode descriptografar as credenciais porque a pasta SYSVOL (\\DOMAIN\SYSVOL\DOMAIN\Policies) é acessível a todos os “Usuários Autenticados” no domínio, o que inclui usuários e computadores. A Microsoft publicou a chave privada [AES private key on MSDN](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN).

## Configuração

- Encryptador de senha para voce escolher a sua propia
[Encryptador online](https://onecompiler.com/ruby/44dww4wg5)

1. **Clic derecho** → **Propiedades** → pestaña **Seguridad**
2. **Botón "Avanzadas"** → pestaña **Auditoría**
3. Edit → Read & Execute
<img width="1715" height="834" alt="w1" src="https://github.com/user-attachments/assets/d191802b-dee6-400f-96a6-e508913dcf2f" />


Groups.xml
```
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}">
  <User clsid="{DF5F1858-51E5-4d24-8B1A-D9BDE98BA1D1}" 
        name="labuser" 
        image="2" 
        changed="2026-02-18 14:30:00" 
        uid="{A1B2C3D4-E5F6-7890-AB12-CD34EF567890}">
    <Properties 
        action="U" 
        fullName="Laboratory User" 
        description="Usuário criado para teste de GPP" 
        cpassword="SENHA_ENCRYPTADA" 
        changeLogon="0" 
        noChange="0" 
        neverExpires="1" 
        acctDisabled="0" 
        userName="labuser"/>
  </User>
  <User clsid="{DF5F1858-51E5-4d24-8B1A-D9BDE98BA1D1}" 
        name="pparker" 
        image="2" 
        changed="2026-02-18 14:35:00" 
        uid="{B2C3D4E5-F6G7-8901-BC23-DE45F6789012}">
    <Properties 
        action="U" 
        fullName="Peter Parker" 
        description="Usuário de domínio com senha em GPP" 
        cpassword="SENHA_ENCRYPTADA" 
        changeLogon="0" 
        noChange="0" 
        neverExpires="1" 
        acctDisabled="0" 
        userName="pparker"/>
  </User>
</Groups>
```

- Habilitacao de script no Powershell
``` PS C:\Users\pparker\Downloads> Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process```
- Caso seja necesario deshabilitar AV e Defender

  ## Ataque

Usaremos a ferramenta [Get-GPPPassword](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1) (Window) e [gpp-decrypt.py](https://github.com/t0thkr1s/gpp-decrypt/blob/master/gpp-decrypt.py)  (linux)
<img width="1596" height="802" alt="finalGPP" src="https://github.com/user-attachments/assets/493eafb1-353a-4940-a9df-79f4bbcb9f67" />

<img width="1088" height="274" alt="gpplinux" src="https://github.com/user-attachments/assets/70be0d71-6af3-435b-acda-10ce66505caf" />

## Prevenção
Simplemente tirar o Group.xml ou tirar as permissoes

## Detecção

- Uma vez que a auditoria esteja ativada, qualquer acesso ao arquivo gerará um evento com o ID 4663
- Eliminar o event id do ossec.conf para que cheguem os logs (por default bem assim)
<img width="845" height="454" alt="w2" src="https://github.com/user-attachments/assets/188241ba-1c27-4ca5-aad5-5acb0eedb530" />

```
<rule id="120007" level="12">
  <if_sid>60103</if_sid>
  <field name="win.system.eventID">^4663$</field>
  <field name="win.eventdata.ObjectName" type="pcre2">(?i).*Policies.*Groups\.xml</field>
  <description>🚨 GPP Attack Detected - Groups.xml accessed by: $(win.eventdata.SubjectUserName)</description>
  <mitre>
    <id>T1552.001</id>
  </mitre>
</rule>
```

- Reiniciar agente e servidor
- Executar GPPDecrypter
  <img width="1586" height="491" alt="w3" src="https://github.com/user-attachments/assets/9750813d-e499-47c2-aa1f-7abfa288c738" />


