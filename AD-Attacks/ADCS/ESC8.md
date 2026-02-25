👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação
- Antes de colocarmos a mão no teclado, é crucial entendermos o que vamos fazer. O ataque segue esta lógica :
- O Alvo: Uma Autoridade Certificadora (CA) com o serviço de "Inscrição Web de Certificados" (CES) ativo e acessível via HTTP (não HTTPS).
- A Falha: Esse endpoint web aceita autenticação NTLM, um protocolo vulnerável a ataques de "relay" ou "retransmissão" .
- O Ataque: Nós, como atacantes (na máquina THEPUNISHER), vamos forçar um servidor privilegiado (o Domain Controller HYDRA-DC) a tentar se autenticar em um servidor malicioso que controlamos.
- O Relay: Em vez de completar a autenticação, nosso servidor malicioso vai "retransmitir" essa tentativa de autenticação para o endpoint web vulnerável do AD CS.
- O Resultado: O AD CS, achando que a solicitação veio do HYDRA-DC, emite um certificado para o próprio HYDRA-DC.
- O Impacto: Com o certificado do Controlador de Domínio, podemos forjar tickets e efetivamente controlar todo o domínio


## Configuração
- Para o ESC8 funcionar, a CA precisa:
  . Ter o papel de "Certification Authority Web Enrollment" instalado. (Provavelmente já está, pois você teve resultados no certipy).
  . Ter o endpoint HTTP (porta 80) habilitado e acessível. Esta é a misconfiguração chave.Aceitar autenticação NTLM neste endpoint (é o comportamento padrão).


Abra o Server Manager no HYDRA-DC → Manage → Add Roles and Features → Clique em Next até chegar em Server Roles → 
Na lista de funções, encontre e expanda (não apenas marque) Active Directory Certificate Services → Agora você verá a lista de Role Services:

  - Certification Authority (já deve estar marcado)
  - Certification Authority Web Enrollment (é este que precisamos marcar) → MARQUE a opção "Certification Authority Web Enrollment" →
      Uma janela pop-up vai aparecer perguntando sobre adicionar recursos necessários (IIS). Clique em Add Features
      Clique em Next até chegar em Confirm installation selections
      Clique em Install
<img width="1421" height="784" alt="config1" src="https://github.com/user-attachments/assets/e13f6896-eb1e-4481-911b-fe6cf3ad5e36" />
  - Click no link e instale 
<img width="1262" height="730" alt="conf2" src="https://github.com/user-attachments/assets/8e963600-adde-4a7e-baad-77ab9f35300b" />


 - Verificar se o IIS Manager Apareceu
  Agora, ao ir em Tools no Server Manager, você deve ver:

  ✅ Internet Information Services (IIS) Manager
<img width="1393" height="715" alt="3" src="https://github.com/user-attachments/assets/0e874b07-e86a-4c50-adc8-b431e6186ea3" />
<img width="1431" height="526" alt="iis1" src="https://github.com/user-attachments/assets/608fde6f-0b92-423d-ab2c-1d0782576642" />

 - Isso aqui quere dizer que esta funcionando!
<img width="962" height="268" alt="4" src="https://github.com/user-attachments/assets/5953ae7a-7e4d-493c-b12f-6259fb25f17f" />

## Ataque
### Recon
```certipy find -u 'fcastle@MARVEL.local' -p 'MYpassword#' -dc-ip 192.168.0.40 -vulnerable -stdout | grep -A 10 "Web Enrollment"```
<img width="676" height="763" alt="5" src="https://github.com/user-attachments/assets/b7b3cb21-dfcb-4cef-a046-ad76512b3c8c" />


Primeiro, ele retransmite a autenticação SMB recebida para a interface de inscrição na Web. Em seguida, ele solicita automaticamente um certificado usando o modelo DomainController. Por fim, após o sucesso, ele armazena o certificado e a chave para uso posterior.

- Forçar o DC a autenticar com o nxc
<img width="1747" height="267" alt="coerceNXC" src="https://github.com/user-attachments/assets/a19d5470-01c3-4a86-a871-b8de5cdce488" />

- Certificado emitido e salvo via ntlmrelayx
<img width="1409" height="559" alt="certificatePFX" src="https://github.com/user-attachments/assets/b25d385f-8071-4921-af8a-7fa287685b99" />
Esta saída confirma que o relé foi bem-sucedido e que a CA emitiu um certificado com ID 20, que é salvo como: hydra-dc.pfx

- Autentique como DC com certificado emitido e extraia o hash do administrador com o DCSync
<img width="1808" height="312" alt="extracthashADMIN" src="https://github.com/user-attachments/assets/39951bac-7ea6-4edf-a480-ea7f7c538fca" />

- Entrando na sessao
<img width="1834" height="802" alt="poc" src="https://github.com/user-attachments/assets/5b0599dd-20d0-4580-a63b-da95f5ad1833" />



## Prevenção
Desative o registro na Web se não for necessário ou restrinja o acesso apenas a usuários internos.
Imponha HTTPS e desative ou restrinja NTLM.
Use autenticação somente Kerberos e defina LmCompatibilityLevel = 5 para recusar NTLMv1.
Fortaleça os modelos de certificado removendo Usuários Autenticados do registro/registro automático e exigindo aprovação do gerente.
Restrinja o acesso à CA e limite as permissões do modelo a grupos privilegiados.
Audite modelos confidenciais, como Controlador de Domínio e Administrador.
Bloqueie vetores de coerção desativando MS-EFSRPC, RPRN, FSRVP e usando o Firewall do Windows.
Habilite os logs de auditoria da CA e monitore as inscrições de certificados de máquina e eventos PKINIT.
Habilite a Proteção Estendida para Autenticação (EPA) para proteger /certsrv no IIS.


## Detecção
Detectar acesso ao endpoint de inscrição web

/etc/suricata/rules/adcs.rules
```alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"ADCS Web Enrollment Access"; flow:to_server,established; content:"GET"; http_method; content:"/certsrv/"; http_uri; classtype:policy-violation; sid:1000001; rev:1;)```

Detectar tentativas de coerção (PetitPotam, Coercer)
```
alert tcp $HOME_NET any -> $HOME_NET 445 (msg:"Potential Coercion - SMB to DC"; flow:to_server; content:"|ff|SMB|2a|"; depth:4; content:"|00 00 00 00|"; distance:4; within:4; classtype:attempted-admin; sid:1000002; rev:1;)

alert tcp $HOME_NET any -> $HOME_NET 135 (msg:"Potential Coercion - RPC to DC"; flow:to_server; content:"|05 00 0b 03 10 00 00 00|"; depth:8; classtype:attempted-admin; sid:1000003; rev:1;)
```

### Wazuh

Detectar coerção via SMB (EventID 5140, 5145)
```
<group name="windows,coercion,">
  <!-- Acesso a pipe nomeado suspeito -->
  <rule id="100012" level="10">
    <if_sid>60006</if_sid>
    <field name="win.system.eventID">5140</field>
    <field name="win.eventdata.ShareName" type="pcre2">(?i)IPC\$</field>
    <field name="win.eventdata.ObjectName" type="pcre2">(?i)lsarpc|efsrpc|spoolss|eventlog</field>
    <description>Possível coerção via pipe nomeado: $(win.eventdata.ObjectName)</description>
    <group>coercion,</group>
  </rule>
</group>
```
```
<!-- /var/ossec/etc/rules/local_rules.xml -->

<group name="windows,adcs,">
  <!-- Solicitação de certificado -->
  <rule id="100010" level="8">
    <if_sid>60006</if_sid>
    <field name="win.system.eventID">4886</field>
    <field name="win.eventdata.Attributes" type="pcre2">(?i)ESC8-Machine|Machine|DomainController</field>
    <description>ADCS: Solicitação de certificado recebida - Template: $(win.eventdata.Template)</description>
    <group>adcs_attack,</group>
  </rule>

  <!-- Certificado emitido para conta privilegiada -->
  <rule id="100011" level="12">
    <if_sid>60006</if_sid>
    <field name="win.system.eventID">4887</field>
    <field name="win.eventdata.Subject" type="pcre2">(?i)HYDRA-DC\$|Administrator|Domain Controllers</field>
    <description>CRÍTICO: Certificado emitido para conta de alto privilégio</description>
    <group>adcs_attack,critical,</group>
  </rule>
</group>
```







