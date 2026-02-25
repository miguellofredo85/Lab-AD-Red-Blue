👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação
O recurso foi lançado inicialmente no Windows Server 2000, mas ainda está presente para compatibilidade com versões anteriores. Basicamente, se um usuário solicitar um ticket de serviço para um serviço em um servidor configurado com delegação irrestrita, esse servidor extrairá o TGT do usuário e o armazenará em cache na memória para uso posterior. Isso significa que o servidor pode se passar por esse usuário em qualquer recurso do domínio.


## Configuração

<img width="1634" height="835" alt="ud1" src="https://github.com/user-attachments/assets/d906db72-5d55-4596-8052-aebee6710337" />

- Exploração da Vulnerabilidade (O Ataque)
O objetivo é fazer um Administrador de Domínio (ou um usuário com altos privilégios) autenticar-se no SPIDERMAN e capturar seu Ticket Granting Ticket (TGT) para fazer um ataque de "Pass-the-Ticket" e acessar o Controlador de Domínio (HYDRA-DC).

- Pré-requisitos no SPIDERMAN (O Computador Vulnerável)
Comprometer o SPIDERMAN: Você precisa de acesso administrativo (SYSTEM) na máquina SPIDERMAN.

## Ataque

- Primeiro achar delegacoes
<img width="1038" height="246" alt="finddelegation-linux" src="https://github.com/user-attachments/assets/18cb4d35-72c5-46c3-85fb-4c4e74fbc63a" />
  
```Import-Module .\PowerView.ps1```
```Get-NetComputer -Unconstrained```
<img width="955" height="155" alt="udfindwin" src="https://github.com/user-attachments/assets/15d63780-b328-41b3-bd4e-50a937fcc546" />


- Ataque
[Coercer para obter o relay](https://github.com/p0dalirius/Coercer)

- Na maquina SPIDERMAN utilizamos Rubeus monitorando por auth
   <img width="1071" height="321" alt="monitor1" src="https://github.com/user-attachments/assets/2fc93616-be78-437c-a95e-725207d2e52e" />

- Coercao e extracao do ticket
  <img width="1918" height="812" alt="exploit" src="https://github.com/user-attachments/assets/f379aed5-1031-42c9-99a1-ac181fc80629" />

- Importacao do ticket e dumping do hash do admin com mimikatz 
<img width="936" height="498" alt="ticketimport" src="https://github.com/user-attachments/assets/e2cf7837-5043-4402-987e-99f9f5391188" />

<img width="776" height="860" alt="dump" src="https://github.com/user-attachments/assets/37737e0f-d68c-4a10-8268-190b632c2332" />

- Entrando no DC com psexec
  <img width="1233" height="970" alt="conecao" src="https://github.com/user-attachments/assets/6fb6ac51-e1c8-44a9-add7-675ae7f76f09" />

- Crackeando NT
  ```miguel~ hashcat -m 1000 -a 0 fc525c9683e8fe067095ba2ddc971889 /opt/SecLists/Passwords/rockyou.txt -O```
  <img width="724" height="502" alt="hash" src="https://github.com/user-attachments/assets/e15e112d-c7a9-4ff8-93a1-955edbb5e0b4" />


## 🛡️ Prevenção

Implementar um firewall RPC de terceiros, como o da Zero Networks, e usá-lo para bloquear funções RPC perigosas. Essa ferramenta também possui um modo de auditoria, permitindo monitorar e obter visibilidade sobre se podem ocorrer interrupções nos negócios ao usá-la ou não. Além disso, ela vai um passo além, fornecendo a funcionalidade de bloquear funções RPC se o OPNUM perigoso associado à coerção estiver presente na solicitação. (Observe que, nessa opção, para cada função RPC recém-descoberta no futuro, teremos que modificar o arquivo de configuração do firewall para incluí-la.)
Bloqueie controladores de domínio e outros servidores de infraestrutura essenciais de se conectarem às portas de saída 139 e 445, exceto para máquinas necessárias para AD (bem como para operações comerciais). Um exemplo é que, embora bloqueemos o tráfego de saída geral para as portas 139 e 445, ainda devemos permiti-lo para controladores de domínio cruzados; caso contrário, a replicação do domínio falhará. (A vantagem dessa solução é que ela também funcionará contra funções RPC vulneráveis recém-descobertas ou outros métodos de coerção.)
- Usar o grupo Protected Users:
Impede autenticação NTLM
Impede delegação (qualquer tipo)
Requer Kerberos com criptografia forte
```Add-ADGroupMember -Identity "Protected Users" -Members "Administrator"```


## 🚨 Detecção
<img width="1851" height="640" alt="wazuhspiderman" src="https://github.com/user-attachments/assets/ef58651e-e6f7-47fb-bd63-cce21610a212" />

```
<group name="windows,security,unconstrained_delegation,">

  <!-- ============================================= -->
  <!-- REGRA 1: FERRAMENTAS DE ATAQUE KERBEROS (GENÉRICA) -->
  <!-- ============================================= -->
  <rule id="110600" level="12">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.CommandLine" type="pcre2">(?i)(Rubeus|mimikatz|SpoolSample|PetitPotam|printerbug|dementor).*</field>
    <description>🚨 Ferramenta de ataque Kerberos detectada: $(win.eventdata.CommandLine)</description>
    <mitre>
      <id>T1558.003</id>
    </mitre>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 2: LOGON DE CONTA DE COMPUTADOR (GENÉRICA) -->
  <!-- ============================================= -->
  <rule id="110601" level="8">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4624$</field>
    <field name="win.eventdata.LogonType">^3$</field>
    <field name="win.eventdata.TargetUserName" type="pcre2">.*\$$</field>
    <description>⚠️ Logon de rede de conta de computador: $(win.eventdata.TargetUserName)</description>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 3: TICKETS KERBEROS SUSPEITOS (GENÉRICA) -->
  <!-- ============================================= -->
  <rule id="110602" level="10">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4769$</field>
    <field name="win.eventdata.TicketOptions" type="pcre2">^0x40800000$|^0x40810000$</field>
    <description>⚠️ Ticket com opções de delegação: $(win.eventdata.ServiceName)</description>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 4: MÚLTIPLOS TICKETS (GENÉRICA) -->
  <!-- ============================================= -->
  <rule id="110603" level="8" frequency="5" timeframe="60">
    <if_matched_sid>60103</if_matched_sid>
    <field name="win.system.eventID">^4769$</field>
    <description>⚠️ Múltiplos tickets Kerberos em curto período</description>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 5: DCSync (QUALQUER CONTA) -->
  <!-- ============================================= -->
  <rule id="110604" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4662$</field>
    <field name="win.eventdata.Properties" type="pcre2">{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}|{19195a5b-6da0-11d0-afd3-00c04fd930c9}</field>
    <description>🚨 DCSync detectado - Conta: $(win.eventdata.SubjectUserName)</description>
    <mitre>
      <id>T1003.006</id>
    </mitre>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 6: MÚLTIPLOS DCSync (GENÉRICA) -->
  <!-- ============================================= -->
  <rule id="110605" level="14" frequency="5" timeframe="60">
    <if_matched_sid>60103</if_matched_sid>
    <field name="win.system.eventID">^4662$</field>
    <description>🚨🚨 Múltiplas operações DCSync em curto período</description>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 7: PRINT SPOOLER (GENÉRICA) -->
  <!-- ============================================= -->
  <rule id="110606" level="10">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^808$</field>
    <description>⚠️ Evento 808 - Possível PrintBug/SpoolSample</description>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 8: ACESSO AO PIPE SPOOLSS -->
  <!-- ============================================= -->
  <rule id="110607" level="8">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^5145$</field>
    <field name="win.eventdata.RelativeTargetName" type="pcre2">(?i).*spoolss.*</field>
    <description>⚠️ Acesso ao pipe spoolss via IPC$</description>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 9: ATIVAÇÃO DE DELEGAÇÃO IRRESTRITA -->
  <!-- ============================================= -->
  <rule id="110608" level="10">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4742$</field>
    <field name="win.eventdata.UserAccountControl" type="pcre2">.*TRUSTED_FOR_DELEGATION.*</field>
    <description>⚠️ Delegação irrestrita ATIVADA para: $(win.eventdata.TargetUserName)</description>
    <mitre>
      <id>T1098</id>
    </mitre>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 10: ENUMERAÇÃO POWERSHELL -->
  <!-- ============================================= -->
  <rule id="110609" level="8">
    <if_group>powershell</if_group>
    <field name="win.eventdata.ScriptBlockText" type="pcre2">(?i)(Get-NetComputer.*-Unconstrained|Get-ADComputer.*TrustedForDelegation)</field>
    <description>⚠️ PowerShell: Enumeração de delegação detectada</description>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 11: CORRELAÇÃO - LOGON + DCSYNC -->
  <!-- ============================================= -->
  <rule id="110610" level="15" frequency="2" timeframe="300">
    <if_matched_sid>110601,110604</if_matched_sid>
    <description>🚨🚨🚨 CRÍTICO: Logon de computador seguido de DCSync - Ataque confirmado</description>
    <options>alert_by_email</options>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 12: CORRELAÇÃO - FERRAMENTA + DCSYNC -->
  <!-- ============================================= -->
  <rule id="110611" level="15" frequency="2" timeframe="600">
    <if_matched_sid>110600,110604</if_matched_sid>
    <description>🚨🚨🚨 CRÍTICO: Ferramenta de ataque seguida de DCSync - Ataque confirmado</description>
    <options>alert_by_email</options>
  </rule>

  <!-- ============================================= -->
  <!-- REGRA 13: CORRELAÇÃO - PRINTBUG + DCSYNC -->
  <!-- ============================================= -->
  <rule id="110612" level="15" frequency="2" timeframe="600">
    <if_matched_sid>110606,110604</if_matched_sid>
    <description>🚨🚨🚨 CRÍTICO: PrintBug seguido de DCSync - Ataque confirmado</description>
    <options>alert_by_email</options>
  </rule>

</group>
```



