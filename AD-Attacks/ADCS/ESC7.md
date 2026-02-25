👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

---

## Explicação
O ESC7 é uma vulnerabilidade crítica de segurança em que os invasores exploram controles de acesso (ACL) fracos dentro das Autoridades Certificadoras (CAs). Ao visar permissões importantes como ManageCA e Manage Certificates, os invasores podem comprometer os sistemas de gerenciamento de certificados. A permissão ManageCA concede controle administrativo, permitindo que invasores modifiquem configurações como EDITF_ATTRIBUTESUBJECTALTNAME2 e explorem vulnerabilidades como a ESC6 usando cmdlets PSPKI. Enquanto isso, a permissão ManageCertificates permite que invasores contornem as verificações de emissão de certificados.

## Configuração

> A permissão “Gerenciar CA” permite que os usuários:
Modifiquem modelos de certificados.
Configurem políticas de emissão de certificados.
Adicionem ou removam agentes de inscrição.
Reiniciem o serviço CA.

<img width="1233" height="610" alt="conf1" src="https://github.com/user-attachments/assets/9d0a3b6b-1f7c-41b4-a667-82ecfbee7cd7" />

## Ataque

> Recon
<img width="946" height="944" alt="scanESC7" src="https://github.com/user-attachments/assets/57ee2227-ffeb-467b-a927-a514a8ca2030" />

> Abuso de ManageCA, adicionando o usuario ckent como Certificate Officer para conseguir certificados em nome de outros
<img width="1362" height="89" alt="addUSER-CA" src="https://github.com/user-attachments/assets/45e8abc4-ba2f-48b6-976b-3f8ff22ce313" />

> Habilitar o Template SubCA que tem privilegios para impersonificar
<img width="1421" height="92" alt="enable-SubCA" src="https://github.com/user-attachments/assets/869d76ba-f874-4fa3-95fe-a13d586b617d" />

> Confirmacao no novo recon ```certipy find -u 'ckent@MARVEL.local' -p 'P@$$w0rd123+' -dc-ip 192.168.0.40 -enabled -stdout```
<img width="860" height="314" alt="enabledca" src="https://github.com/user-attachments/assets/1069fe53-ab83-48c8-9d2c-87e3ecb989f4" />

> Agora vamos baixar  o certificado
<img width="1918" height="272" alt="1" src="https://github.com/user-attachments/assets/af9514f0-6718-43ec-862d-7e477b3c66f5" />

> Pedir e baixar o pfx
<img width="1272" height="314" alt="2" src="https://github.com/user-attachments/assets/fc11c492-149b-4fc5-af5a-787c4c674e89" />

> Dumping do NT Admin
<img width="1910" height="186" alt="dumpntadmin" src="https://github.com/user-attachments/assets/079a7f7d-d4cf-4290-bd38-e35551c6f450" />


## Prevenção
- Restringir privilégios de gerenciamento (ManageCA e ManageCertificates)
- Separação de funções
- Restringir o “Enrollment”
- Aprovação do gerente de certificados

## Detecção
monitorar o ID de evento 4890 (Autoridade de certificação alterada) e 4899 (Modelo alterado).

```
<group name="ad_cs_attacks,">
  <rule id="100500" level="12">
    <if_sid>60103</if_sid> <field name="win.system.eventID">^4890$</field>
    <description>AD CS: Se han modificado los permisos de la CA (Posible ESC7/ManageCA)</description>
    <mitre>
      <id>T1557.001</id>
    </mitre>
  </rule>

  <rule id="100501" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4871$</field>
    <field name="win.eventdata.subjectUserName" type="pcre2">^(?!.*\$).*$</field>
    <description>AD CS: Solicitud de certificado aprobada manualmente por un Officer</description>
  </rule>
</group>
```
