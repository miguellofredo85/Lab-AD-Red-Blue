👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)


## Explicação
- Explora um erro de fortalecimento baseado no registro (DisableExtensionList).
- Aproveita as permissões no nível de atributo (acesso de gravação ao UPN).
- Permite persistência furtiva por meio de credenciais ocultas.
- Funciona mesmo em ambientes fortalecidos com StrongCertificateBindingEnforcement habilitado.
- Resultado: um comprometimento completo do Active Directory com pontos de contato mínimos.
- O domínio deve ter os Serviços de Certificados do Active Directory e a Autoridade Certificadora configurados.


## Configuração

<img width="1586" height="562" alt="1" src="https://github.com/user-attachments/assets/89e26dec-eabb-4254-9e5a-719ebe8f0733" />

<img width="1764" height="827" alt="configkeycredential" src="https://github.com/user-attachments/assets/4bd70fd5-18b6-4b24-ab0f-5342c00efba9" />

<img width="1246" height="754" alt="writeconfig" src="https://github.com/user-attachments/assets/a0e630c7-dbdf-4b44-86f3-3fb826e63904" />

<img width="1529" height="533" alt="tstarckgroups" src="https://github.com/user-attachments/assets/b8a70a66-ee89-455f-88cc-2668622ae49e" />


## Ataque

Scan vulnerabilidades de certificados
```
ertipy find -vulnerable -u fcastle@MARVEL.local -p 'MYpassword#' -stdout
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 18 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'MARVEL-HYDRA-DC-CA' via RRP
[*] Successfully retrieved CA configuration for 'MARVEL-HYDRA-DC-CA'
[*] Checking web enrollment for CA 'MARVEL-HYDRA-DC-CA' @ 'HYDRA-DC.MARVEL.local'
[!] Error checking web enrollment: [Errno 111] Connection refused
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : MARVEL-HYDRA-DC-CA
    DNS Name                            : HYDRA-DC.MARVEL.local
    Certificate Subject                 : CN=MARVEL-HYDRA-DC-CA, DC=MARVEL, DC=local
    Certificate Serial Number           : 589AAD803FD4FFA24262A85B572A3DB2
    Certificate Validity Start          : 2026-02-04 02:30:14+00:00
    Certificate Validity End            : 2125-02-04 02:40:14+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Disabled Extensions                 : 1.3.6.1.4.1.311.25.2                                  
    Permissions
      Owner                             : MARVEL.LOCAL\Administrators
      Access Rights
        ManageCa                        : MARVEL.LOCAL\Administrators
                                          MARVEL.LOCAL\Domain Admins
                                          MARVEL.LOCAL\Enterprise Admins
        ManageCertificates              : MARVEL.LOCAL\Administrators
                                          MARVEL.LOCAL\Domain Admins
                                          MARVEL.LOCAL\Enterprise Admins
        Enroll                          : MARVEL.LOCAL\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
      ESC16                             : Security Extension is disabled.
    [*] Remarks
      ESC16                             : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
```

## Recon com bloodhound
<img width="1846" height="744" alt="pathtodomain" src="https://github.com/user-attachments/assets/f4ed1f61-595d-4300-bd6a-f2080e7204d6" />

- Modificacao do upn
<img width="1487" height="131" alt="upnchanged" src="https://github.com/user-attachments/assets/c1ed1539-a578-4991-b802-cfc634574835" />

- Obtendo o hash NT do administrator
<img width="1280" height="484" alt="tstarkhash" src="https://github.com/user-attachments/assets/b8725c43-070c-422e-866e-0b2ee894e5aa" />

- Obtendo o pfx
<img width="946" height="343" alt="pfx" src="https://github.com/user-attachments/assets/237543a7-c3a3-4d3e-b026-e86da56983a4" />

- Dumpe de hashes
<img width="1398" height="275" alt="dump" src="https://github.com/user-attachments/assets/6c9a2f1c-9d93-404b-967a-b344900c1016" />

## Prevenção
Restringir permissões de modelo de certificado → Somente usuários privilegiados devem ter direitos de inscrição.
Aplicar criptografia forte → Use RSA 3072/4096 bits e SHA-256/SHA-512.
Desativar atributos SAN definidos pelo usuário → Impedir a falsificação de identidade não autorizada.
Monitorar a emissão de certificados → Habilite a auditoria para os IDs de evento 4886, 4887, 4768.
Implementar políticas de revogação de certificados → Use CRLs e OCSP para invalidar certificados roubados.

## Detecção

- Quando você usa certipy auth com um certificado, o Domain Controller gera um Evento 4768 com PreAuthType = 16 (PKINIT). Isso é inevitável — o atacante precisa fazer isso para obter o TGT
```
<!-- /var/ossec/etc/rules/local_rules.xml -->
<group name="windows,kerberos,pkinit,">
  <rule id="100030" level="12">
    <if_sid>60006</if_sid>
    <field name="win.system.eventID">4768</field>
    <field name="win.eventdata.PreAuthType">16</field>
    <field name="win.eventdata.TargetUserName" type="pcre2">^(?!.*\$$).*$</field> <!-- Exclui contas de computador -->
    <description>🚨 Autenticação PKINIT (certificado) detectada para $(win.eventdata.TargetUserName)</description>
    <group>pkinit,critical,certificate_auth,</group>
  </rule>
</group>
```

- Falhas de autenticação PKINIT (tentativas com certificado inválido)
```
<group name="windows,kerberos,pkinit,failure,">
  <rule id="100031" level="8" frequency="5" timeframe="300">
    <if_matched_sid>60006</if_matched_sid>
    <field name="win.system.eventID">4771</field>
    <field name="win.eventdata.PreAuthType">16</field>
    <description>⚠️ Múltiplas falhas de autenticação PKINIT - Possível tentativa de ataque com certificado</description>
    <group>pkinit,bruteforce,</group>
  </rule>
</group>  
```

