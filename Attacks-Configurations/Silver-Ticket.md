O Silver Ticket é um bilhete de serviço (TGS) forjado que permite acessar um serviço específico (como CIFS, HTTP, LDAP) sem nunca contactar o Domain Controller . Diferente do Golden Ticket (que forja TGTs e dá acesso a tudo), o Silver Ticket é focado e silencioso porque:

Não gera logs no Domain Controller (o DC nem sabe que o ticket foi usado) 

A validação acontece apenas no servidor de destino

O serviço alvo aceita o ticket porque consegue descriptografá-lo com seu próprio hash

##  🔧 Como funciona o ataque?
1. AS-REQ: Usuário → KDC (pede TGT)
2. AS-REP: KDC → Usuário (entrega TGT, criptografado com hash do krbtgt)
3. TGS-REQ: Usuário → KDC (pede ticket para serviço, apresentando TGT)
4. TGS-REP: KDC → Usuário (entrega TGS, criptografado com hash do serviço)
5. AP-REQ: Usuário → Serviço (apresenta TGS)
6. Serviço valida o ticket (consegue descriptografar com seu próprio hash)

## 🏗️ Configurando o laboratório para Silver Ticket

Identificar um serviço com SPN registrado (qualquer serviço com conta associada):
```setspn -Q */*```

### Contas de servicio alvo
<img width="1166" height="565" alt="lm" src="https://github.com/user-attachments/assets/afb640cf-b9fb-4c2b-a95b-4a6683cb68c6" />




### O Ataque Silver Ticket
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


### SID
<img width="1679" height="94" alt="DomainSid" src="https://github.com/user-attachments/assets/49c267c5-30b6-4137-aff9-f35987ad4459" />

### SPN
<img width="1714" height="478" alt="SPN" src="https://github.com/user-attachments/assets/bfacbf4d-bc18-4639-a121-a0fb63ee1141" />

Com hashcat crackeamos o hash do usuario SQLService que no nosso exemplo e kerberosteable
```hashcat -a 0 -m 13100 mssql.hash /opt/SecLists/Passwords/rockyou.txt -O```
```$krb5tgs$23$*SQLService$MARVEL.LOCAL$MARVEL.local/SQLService*$ba2ef8e570680b9c43cc99c......:Goku123!```

### Hasheamos a senha
[NT HASHER](https://www.browserling.com/tools/ntlm-hash)
```E4933A6CCF60591C5C1AC40C6A3DB382```

### Criacao do ticket com os dados que recoletamos
<img width="1722" height="357" alt="silver-ticket-generated" src="https://github.com/user-attachments/assets/bb8fc0f7-70c0-4da8-bc69-6ffb73c906b1" />

### Exportar o ticket para o ambiente e verificamos que esteja forjado
<img width="944" height="185" alt="ccache-file" src="https://github.com/user-attachments/assets/f0d410aa-3a12-413b-992e-9342181cad5d" />


Como temos salvo o ticket no Administrator.ccache nao precisamos colocal a pass para autenticarnos no servicio
```python3 /opt/impacket-0-13-0/examples/mssqlclient.py  -k -no-pass MARVEL.local -windows-auth -p 60111```




















