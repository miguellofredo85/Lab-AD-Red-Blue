👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

---

## Explicação
Extended Rights (Direitos Estendidos) são permissões especiais no Active Directory que vão além das permissões padrão de "Ler" e "Escrever". Elas controlam operações específicas e sensíveis no AD .
---
Cada Extended Right é identificado por um GUID único e está associado a uma operação específica 

## Configuração 
> NO DC
> ATIVAR A LIXEIRA DO AD (RECYCLE BIN)
1. Server Manager -> Tools -> Active Directory Administrative Center (ADAC)
2. No painel esquerdo, selecione o domínio MARVEL.local
3. No painel Tasks à direita, clique em "Enable Recycle Bin"
4. Confirme a mensagem de aviso (ação irreversível)
5. Aguarde a replicação e pressione F5 para atualizar

> CONFIGURAR PERMISSÕES PARA O USUÁRIO RESTAURAR OBJETOS
1. Server Manager -> Tools -> Users and Computers
2. click direito no MARVEL.local -> Properties
3. Advanced -> Edit -> Change Principal (escolha o usuario)
4. Na lista de permissoes procure no Permissions pelo Reanimate Tombstone, check e ok.
<img width="1661" height="750" alt="addTombstonetoFCASTLE" src="https://github.com/user-attachments/assets/312f81f9-a64b-43ef-a82e-3a65f9c7f37b" />

> No HYDRA-DC, abra o Prompt de Comando (não PowerShell) como administrador.
. ```dsacls "CN=Deleted Objects,DC=MARVEL,DC=local" /takeownership``` Tomar posse do container Deleted Objects
. ```dsacls "CN=Deleted Objects,DC=MARVEL,DC=local" /G "MARVEL\fcastle:LCRP"``` Conceder permissão de leitura no container Deleted Objects
. ```dsacls "CN=Users,DC=MARVEL,DC=local" /G "MARVEL\fcastle:CC"``` Conceder permissão de criação na OU de destino (Users) 

> Explicação dos parâmetros :
. CA = Control Access (direito de controle especial)
. LCRP = List Contents + Read Property
. CC = Create Child (criar objetos filhos)

> CRIAR E EXCLUIR UM USUÁRIO PARA TESTE
. No HYDRA-DC (PowerShell como administrador):
```
# Criar um usuário de teste
$novoGuid = New-ADUser -Name "fuser_teste" `
    -SamAccountName "fuser_teste" `
    -UserPrincipalName "fuser_teste@MARVEL.local" `
    -GivenName "F" `
    -Surname "User" `
    -DisplayName "F User Test" `
    -Enabled $true `
    -AccountPassword (ConvertTo-SecureString "Senha123!" -AsPlainText -Force) `
    -PassThru | Select-Object -ExpandProperty ObjectGUID

Write-Host "Novo GUID: $novoGuid" -ForegroundColor Green

# Excluir o usuário (vai para a lixeira)
Remove-ADUser -Identity $novoGuid -Confirm:$false

# Confirmar que está na lixeira
Get-ADObject -Identity $novoGuid -IncludeDeletedObjects -Properties * | 
    Format-List Name, SamAccountName, IsDeleted, LastKnownParent, DistinguishedName
```
<img width="1538" height="473" alt="PoCDeletedUser" src="https://github.com/user-attachments/assets/f2c9b708-1745-4395-9427-2b6b521cd638" />




> RESTAURAR O USUÁRIO PELA MÁQUINA CLIENTE
> Restaurar pelo DistinguishedName
. ```Restore-ADObject -Identity "CN=fuser_teste\0ADEL:seu-guid-aqui,CN=Deleted Objects,DC=MARVEL,DC=local" -Server HYDRA-DC.MARVEL.local```
> Verificar se foi restaurado
. ```Get-ADUser fuser_teste -Server HYDRA-DC.MARVEL.local | Format-List Name, SamAccountName, Enabled, DistinguishedName```
<img width="1697" height="285" alt="restore-user" src="https://github.com/user-attachments/assets/6c369f96-c6b3-47b2-8a91-5d0272d9e565" />
---

## Ataque
> Procuraremos por usuarios/grupos que possam ter extended rights, no nosso caso aqui sera do nosso usuario fcastle mesmo
<img width="797" height="102" alt="convertsid" src="https://github.com/user-attachments/assets/06c964e7-3cdd-4f39-9456-51bcfcd9957b" />




> Ver objetos deletado na lixeira
```
Get-ADObject -SearchBase "CN=Deleted Objects,DC=MARVEL,DC=local" -Filter "objectClass -eq 'user'" -IncludeDeletedObjects -Server HYDRA-DC.MARVEL.local | Select-Object Name, LastKnownParent, ObjectGUID | Format-Table -AutoSize
```
<img width="1694" height="710" alt="Get-ADObject" src="https://github.com/user-attachments/assets/68de6f9d-d385-4c1d-af39-ef7dc3f4a633" />


## Prevenção
...

## Detecção
...
