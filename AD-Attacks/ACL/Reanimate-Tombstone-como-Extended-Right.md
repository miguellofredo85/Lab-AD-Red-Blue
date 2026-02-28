👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

---

## Explicação

Extended Rights (Direitos Estendidos) são permissões especiais no Active Directory que vão além das permissões padrão de "Ler" e "Escrever". Elas controlam operações específicas e sensíveis no AD 
Cada Extended Right é identificado por um GUID único e está associado a uma operação específica.

---
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

> guid sem resolver
<img width="1451" height="543" alt="s guisnotresolve" src="https://github.com/user-attachments/assets/d81be6dc-7f17-4446-8455-0436661a20bf" />

> 
<img width="1554" height="441" alt="ResolveGUIDs" src="https://github.com/user-attachments/assets/88ff9deb-a83f-4aeb-88a2-65286cd2944a" />


> Ver objetos deletado na lixeira
```
 Get-ADObject -Filter 'IsDeleted -eq $true' -IncludeDeletedObjects -Properties * -Server HYDRA-DC.MARVEL.local | Select-Object Name, ObjectClass, LastKnownParent, ObjectGUID | Format-Table -AutoSize
```
<img width="1617" height="245" alt="muitosguids" src="https://github.com/user-attachments/assets/6258219e-935d-4c22-8b2c-cb7e7c0f9fe3" />



> Restore do user
<img width="1870" height="668" alt="restoreuser" src="https://github.com/user-attachments/assets/f006ec49-30f9-43d6-926c-0f6e9be4738d" />

---
## Prevenção
1. PRINCÍPIO DO MENOR PRIVILÉGIO (POLE)
Ninguém deve ter mais permissões do que o necessário para sua função.

2. SEGREGAÇÃO DE FUNÇÕES (SoD)
Quem restaura não deveria ser quem gerencia usuários do dia a dia.

3. PROTEÇÃO POR CAMADAS (Defense in Depth)
Não confie em uma única medida de segurança.

```
# Criar um grupo específico para restaurações
New-ADGroup -Name "AD_Restore_Operators" -GroupScope Universal -GroupCategory Security -Description "Autorizados a restaurar objetos do AD"

# Adicionar apenas usuários que PRECISAM restaurar
Add-ADGroupMember -Identity "AD_Restore_Operators" -Members "fcastle"

# Conceder a permissão APENAS para este grupo (não para usuários individuais)
dsacls "DC=MARVEL,DC=local" /G "MARVEL\AD_Restore_Operators:CA;Reanimate Tombstones"
```

...

## Detecção
Event ID 5136	DC (Segurança)	Modificação de objeto no AD (incluindo permissões)
Event ID 4670	DC (Segurança)	Permissões alteradas em um objeto

<!-- /var/ossec/etc/rules/local_rules.xml -->
<group name="windows,active_directory,">

  <!-- Regra 1: Detectar restauração de objeto (Reanimate Tombstone) -->
  <rule id="100001" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^5136$</field>
    <field name="win.eventdata.OperationType">^Value Added$</field>
    <field name="win.eventdata.AttributeLDAPDisplayName">^isRecycled$</field>
    <description>AD Recycle Bin: Objeto restaurado (Reanimate Tombstone) - Possível abuso de Extended Right</description>
    <group>ad_recycle,ad_attack,</group>
    <mitre>
      <id>T1098</id> <!-- Account Manipulation -->
    </mitre>
  </rule>

  <!-- Regra 2: Detectar mudança de permissão em objeto do AD -->
  <rule id="100002" level="10">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4670$</field>
    <description>Permissões alteradas em objeto do AD - Verificar possível delegação maliciosa</description>
    <group>ad_permission_change,ad_attack,</group>
    <mitre>
      <id>T1222</id> <!-- File and Directory Permissions Modification -->
    </mitre>
  </rule>

  <!-- Regra 3: Detectar acesso a objeto com controle especial -->
  <rule id="100003" level="8">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4662$</field>
    <field name="win.eventdata.Properties">.*Reanimate Tombstones.*</field>
    <description>Acesso a objeto com direito estendido: Reanimate Tombstones</description>
    <group>ad_extended_right_access,ad_recon,</group>
  </rule>

  <!-- Regra 4: Detectar ferramentas de AD sendo executadas -->
  <rule id="100004" level="7">
    <if_sid>61603</if_sid> <!-- Sysmon ID 1 -->
    <field name="win.eventdata.Image">.*PowerView.*|.*ADModule.*|.*SharpView.*|.*ntdsutil.*|.*dsacls.*</field>
    <description>Ferramenta de AD potencialmente maliciosa detectada: $(win.eventdata.Image)</description>
    <group>ad_tool_execution,</group>
    <mitre>
      <id>T1059</id> <!-- Command and Scripting Interpreter -->
    </mitre>
  </rule>

  <!-- Regra 5: Detectar quem fez restore e de onde -->
  <rule id="100005" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^5136$</field>
    <field name="win.eventdata.AttributeLDAPDisplayName">^isRecycled$</field>
    <field name="win.eventdata.SubjectUserName">^((?!SYSTEM|DC$).)*$</field>
    <description>Usuário $(win.eventdata.SubjectUserName) restaurou objeto no AD a partir de $(win.eventdata.WorkstationName)</description>
    <group>ad_recycle,ad_forensics,</group>
  </rule>

</group>
