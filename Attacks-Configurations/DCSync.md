## Descricao
O DCSync é aquele ataque onde os caras se passam por um Controlador de Domínio e pedem replicação pra um Controlador de Domínio de verdade, tudo pra roubar os hashes de senha do Active Directory. Esse ataque pode ser feito tanto por uma conta de usuário quanto por um computador, desde que eles tenham as permissões certas, que são:

Replicating Directory Changes

Replicating Directory Changes All

## Config
- No DC
Server Manager Dashboard -> Tools -> Users and Computers -> click direito MARVEL.local -> Properties -> Add (coloca o usuario) -> check boxes -> apply
<img width="1731" height="741" alt="config1posta" src="https://github.com/user-attachments/assets/c34871b7-adca-4958-bbb6-e07a3ec7d163" />

## Attack

- Desde Windows com mimikatz
<img width="1015" height="836" alt="attackmimikatz" src="https://github.com/user-attachments/assets/86d4cdb1-23ea-4eab-b66d-0113dd07de4c" />

- No Linux com secretdump de impacket ou netexec
<img width="1204" height="625" alt="fromlinux" src="https://github.com/user-attachments/assets/9e4260d9-2be6-4922-a593-f81bb93385b8" />

<img width="1539" height="406" alt="ntdsnxc" src="https://github.com/user-attachments/assets/24742dc8-0efc-4d77-92a2-e7b86aa023ac" />




## Prevencao
- Uncheck replication para usuarios
- Evitar autenticacao pela red
- Aplicar [RPC Firewall](https://github.com/zeronetworks/rpcfirewall) para bloquear chamadas por RPC

## Wazuh log e regra
<img width="1918" height="280" alt="dashhydrawazuh" src="https://github.com/user-attachments/assets/5b49edd4-6397-4d2e-8f7f-6c7d072f7329" />


```
    <!-- DETECÇÃO DCSYNC - Baseado no Event ID 4662 -->
<rule id="110017" level="12">
  <if_sid>60103</if_sid>  <!-- Grupo de eventos de segurança do Windows -->
  <field name="win.system.eventID">^4662$</field>
  <field name="win.eventdata.properties" type="pcre2">{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}|{19195a5b-6da0-11d0-afd3-00c04fd930c9}</field>
  <field name="win.eventdata.SubjectUserName" negate="yes">\$$</field> <!-- Ignora contas de computador (que terminam com $) -->
  <description>🚨 Possível ataque DCSync detectado - Conta: $(win.eventdata.SubjectUserName) tentou replicação de diretório</description>
  <mitre>
    <id>T1003.006</id> <!-- ATT&CK para DCSync / Credential Dumping -->
  </mitre>
</rule>
```

