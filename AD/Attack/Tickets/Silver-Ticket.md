👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação
O Silver Ticket é um bilhete de serviço (TGS) forjado que permite acessar um serviço específico (como CIFS, HTTP, LDAP) sem nunca contactar o Domain Controller . Diferente do Golden Ticket (que forja TGTs e dá acesso a tudo), o Silver Ticket é focado e silencioso porque:

Não gera logs no Domain Controller (o DC nem sabe que o ticket foi usado) 

A validação acontece apenas no servidor de destino

O serviço alvo aceita o ticket porque consegue descriptografá-lo com seu próprio hash

###  🔧 Como funciona o ataque?
1. AS-REQ: Usuário → KDC (pede TGT)
2. AS-REP: KDC → Usuário (entrega TGT, criptografado com hash do krbtgt)
3. TGS-REQ: Usuário → KDC (pede ticket para serviço, apresentando TGT)
4. TGS-REP: KDC → Usuário (entrega TGS, criptografado com hash do serviço)
5. AP-REQ: Usuário → Serviço (apresenta TGS)
6. Serviço valida o ticket (consegue descriptografar com seu próprio hash)

## 🏗️ Configuração

Identificar um serviço com SPN registrado (qualquer serviço com conta associada):
```setspn -Q */*```
Configurar segundo [MSSQL Server Configuracao](https://github.com/miguellofredo85/Lab-AD-Red-Blue/blob/main/AD-Config/MSSQL.md)

### Contas de servicio alvo
<img width="1166" height="565" alt="lm" src="https://github.com/user-attachments/assets/afb640cf-b9fb-4c2b-a95b-4a6683cb68c6" />




## Ataque
O atacante pula as etapas 1-4 e vai direto para a etapa 5, criando ele mesmo um TGS falso. 
Para isso precisa:
├── 📛 Nome do domínio: ex: "lab.local" 
├── 🆔 SID do domínio: ex: "S-1-5-21-3737340914-1718799860-1234567890" (nxc ldap 192.168.0.40 -u pparker -p 'Password2!' --get-sid)
├── 🔑 Hash NTLM da conta do serviço alvo: "E4933A6CCF60591C5C1AC40C6A3DB382" (se voce souber a senha do svc_mssql por exemplo no nosso caso e Goku123! [NT HASHER](https://www.browserling.com/tools/ntlm-hash))
├── 🎯 SPN do serviço: ex: "cifs/dc.lab.local" ou "http/www.lab.local" 
└── 👤 Usuário falso: qualquer nome (ex: "administrator")

- O servidor de serviço só verifica se consegue descriptografar o ticket com seu próprio hash
- Ele não contacta o DC pra validar se o ticket foi realmente emitido
- Se você tem o hash do serviço, pode criar tickets que o serviço aceitará como legítimos

### De momento sem privilegios
<img width="1370" height="505" alt="DENEGADO-MSSQL" src="https://github.com/user-attachments/assets/cedf82c7-02e8-42e4-a4cc-6925124dc08f" />

### SID
<img width="1679" height="94" alt="DomainSid" src="https://github.com/user-attachments/assets/49c267c5-30b6-4137-aff9-f35987ad4459" />

### SPN
<img width="1714" height="478" alt="SPN" src="https://github.com/user-attachments/assets/bfacbf4d-bc18-4639-a121-a0fb63ee1141" />

Com hashcat crackeamos o hash do usuario SQLService que no nosso exemplo e kerberosteable
```hashcat -a 0 -m 13100 mssql.hash /opt/SecLists/Passwords/rockyou.txt -O```
```$krb5tgs$23$*SQLService$MARVEL.LOCAL$MARVEL.local/SQLService*$ba2ef8e570680b9c43cc99c......:Goku123!```

Ou pudemos enviar o NTLMv2 mediante autenticacion por xp_dirtree e crackear o ntlm se puder
<img width="1371" height="166" alt="dt1" src="https://github.com/user-attachments/assets/e86e17e8-4db1-4c66-ab3f-4e78660f5392" />
<img width="1100" height="226" alt="dt2hash" src="https://github.com/user-attachments/assets/3a658338-0b6c-43c3-bb39-dbe28630e401" />


### Hasheamos a senha
[NT HASHER](https://www.browserling.com/tools/ntlm-hash)
```E4933A6CCF60591C5C1AC40C6A3DB382```

### Criacao do ticket com os dados que recoletamos
<img width="1866" height="364" alt="ticketadmin" src="https://github.com/user-attachments/assets/8cb6b62d-2657-441a-9e41-a8d01ed567d1" />

### Exportar o ticket para o ambiente e verificamos que esteja forjado
```export KRB5CCNAME=Administrator.ccache```
```
klist
Ticket cache: FILE:Administrator.ccache
Default principal: Administrator@MARVEL.LOCAL

Valid starting       Expires              Service principal
02/19/2026 04:34:18  02/17/2036 04:34:18  MSSQLSvc/HYDRA-DC.MARVEL.local:60111@MARVEL.LOCAL
	renew until 02/17/2036 04:34:18

```

Como temos salvo o ticket no Administrator.ccache nao precisamos colocal a pass para autenticarnos no servicio
<img width="1467" height="494" alt="auth" src="https://github.com/user-attachments/assets/87da5b07-a90e-4b50-8b0d-d05e9740d7d4" />

## 🛡️ Prevenção
- Use contas gerenciadas (gMSA) que trocam senhas automaticamente
- Limite contas com SPNs - só o necessário
- Mude senhas regularmente (invalida Silver Tickets existentes)
- Desabilitar RC4 (forçar AES)
- Contas de serviço não precisam ser Domain Admin

## Detecção

Adicionar no ossec.conf do dc
```
<localfile>
  <location>C:\Program Files\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQL\Log\ERRORLOG</location>
  <log_format>syslog</log_format>
</localfile>
```

Regra 
```
<group name="silver_ticket,recon,kerberos,">

  <!-- Regra 14: Conexões para porta SQL anômalas -->
  <rule id="140014" level="7">
    <if_group>suricata</if_group>
    <field name="dest_port">^60111$</field>
    <field name="src_ip" negate="yes">^192\.168\.0\.40</field> <!-- IPs autorizados -->
    <description>⚠️ Conexão SQL de origem não autorizada: $(src_ip) para porta $(dest_port)</description>
    <mitre>
      <id>T1558.003</id>
    </mitre>
  </rule>
<rule id="140007" level="12">
    <if_sid>60103</if_sid> <!-- Grupo Security Events -->
    <field name="win.system.eventID">^4769$</field> <!-- Kerberos Service Ticket Request -->
    <field name="win.eventdata.ServiceName" type="pcre2">^(?i)(cifs|host|http|mssqlsvc|ldap)/HYDRA-DC\.MARVEL\.local</field>
    <field name="win.eventdata.TicketOptions">^0x40800000$|^0x40810000$</field> <!-- S4U2Proxy flags -->
    <field name="win.eventdata.TargetUserName" negate="yes">.*\$$</field> <!-- Ignorar contas de computador -->
    <description>⚠️ Possível Silver Ticket: Requisição de ticket para serviço crítico de $(win.eventdata.IpAddress) - Usuário: $(win.eventdata.TargetUserName)</description>
    <mitre>
      <id>T1558.003</id>
    </mitre>
  </rule>
</group>
```

<img width="1897" height="390" alt="wazuhST" src="https://github.com/user-attachments/assets/12dce797-9d7a-4d82-8ca8-8a48cf30372b" />







