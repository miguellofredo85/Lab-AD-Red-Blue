👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação
ESC2 (Escalation Path 2) é uma vulnerabilidade nos Serviços de Certificados do Active Directory (AD CS) em que um modelo de certificado permite que usuários com privilégios limitados se inscrevam, e o modelo inclui usos de chave estendidos (EKUs) perigosos, como:

Autenticação do cliente (1.3.6.1.5.5.7.3.2)
Logon com cartão inteligente (1.3.6.1.4.1.311.20.2.2)
Qualquer finalidade (2.5.29.37.0)
Esses EKUs permitem que o invasor solicite um certificado e se autentique como um usuário diferente por meio do Kerberos (PKINIT), ignorando completamente as senhas.

## Configuração

### Usuario
1. No HYDRA-DC, abra "Usuários e Computadores do Active Directory".
2. Encontre o usuário ckent.
3. Clique com botão direito → "Propriedades".
4. Vá para a guia "Geral".
5. No campo "Email", adicione um endereço qualquer, como usuario@marvel.local.
6. Forzar replicacao no powershell ``` repadmin /syncall /AdeP```

### Template
1. Abra o Console de Modelos de Certificado:
  - Pressione Win + R, digite certtmpl.msc e pressione Enter. Isso abrirá o gerenciador de modelos de certificado
2. Click direito sobre "User" -> Duplicar Template
3. Aba Subject Name -> UPN check e Email name unchek | Supply in the request checked
4. Aba Request Handling -> Allow private key to be exported check
5. Aba Security -> Domain Users enroll  
6. Aba Extensions:
  - Remova Email
  - Opção B1 (Recomendada para simplicidade): Clique em Remove em cada uma das políticas listadas até que a lista fique vazia. Um template sem nenhum EKU é tão vulnerável quanto um com "Any Purpose" . Clique em OK para fechar.

  - Opção B2 (Mais explícita): Clique em Add. No campo "Nome da política", digite "Any Purpose" e no campo "Identificador de objeto", digite 2.5.29.37.0 . Clique em OK e depois em OK novamente na janela de edição.
<img width="1557" height="828" alt="esc2-1" src="https://github.com/user-attachments/assets/b19a9455-96a1-4973-a763-edfb4a376871" />
7. Publicar o Template na Autoridade de Certificação, pressione Win + R, digite certsrv.msc
8. MARVEL-HYDRA-DC-CA -> Certificate Templates -> click direito New -> adicionar seu template criado
<img width="1670" height="643" alt="esc2-2" src="https://github.com/user-attachments/assets/0e2fae05-b91c-41b6-9011-e5e722049da6" />
9. Reiniciar DC 
10. Recon 
```
certipy find -vulnerable -u ckent@MARVEL.local -p 'P@$$w0rd123+' -stdout
 Template Name                       : ESC2-User
    Display Name                        : ESC2-User
    Certificate Authorities             : MARVEL-HYDRA-DC-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : True
    Any Purpose                         : True
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectAltRequireEmail
                                          SubjectRequireEmail
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Secure Email
                                          Encrypting File System
                                          Client Authentication
                                          Any Purpose
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2026-02-25T07:48:26+00:00
    Template Last Modified              : 2026-02-25T07:57:18+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : MARVEL.LOCAL\Domain Admins
                                          MARVEL.LOCAL\Domain Users
                                          MARVEL.LOCAL\Enterprise Admins
      Object Control Permissions
        Owner                           : MARVEL.LOCAL\Administrator
        Full Control Principals         : MARVEL.LOCAL\Domain Admins
                                          MARVEL.LOCAL\Enterprise Admins
        Write Owner Principals          : MARVEL.LOCAL\Domain Admins
                                          MARVEL.LOCAL\Enterprise Admins
        Write Dacl Principals           : MARVEL.LOCAL\Domain Admins
                                          MARVEL.LOCAL\Enterprise Admins
        Write Property Enroll           : MARVEL.LOCAL\Domain Admins
                                          MARVEL.LOCAL\Domain Users
                                          MARVEL.LOCAL\Enterprise Admins
    [+] User Enrollable Principals      : MARVEL.LOCAL\Domain Users
    [!] Vulnerabilities
      ESC2                              : Template can be used for any purpose.
```
<img width="1670" height="643" alt="esc2-2" src="https://github.com/user-attachments/assets/b9a2ca79-1a2e-4cdf-b67c-7f29f5de5ecb" />


## Ataque
- Solicitar TGT e salvar como .cache
  <img width="1853" height="546" alt="ccache" src="https://github.com/user-attachments/assets/24d3d966-64c2-4679-b33d-a0e05c767577" />


<img width="1123" height="312" alt="upns" src="https://github.com/user-attachments/assets/4566f286-9c65-4f40-9691-a4a17b8dc19c" />

> upn sao sempre nome-de-usuario@dominio.local


<img width="746" height="670" alt="sids" src="https://github.com/user-attachments/assets/063d66a2-7358-4c94-ab05-ecc56e798f1f" />

> sid de usuarios, o admin sempre e 500


- Salvar o ticket e dumpear credenciais
  <img width="1184" height="278" alt="dump" src="https://github.com/user-attachments/assets/ee07dcc1-a372-47a9-ae9f-f38d02e9b631" />

- Ou usar nxc com o arquivo admin.pfx para dumpear
  <img width="1855" height="156" alt="nxcPFX" src="https://github.com/user-attachments/assets/7a87a69e-3b85-4223-890d-edee5b8e7246" />



## Prevenção
- Restringir Permissões de Inscrição (Enrollment Rights)
- Restringir Permissões de Controle (Write/Full Control)
- Configuração Segura de Templates
- Habilitar a Aprovação do Gerente da CA

 
## Detecção
Monitorar Eventos de Solicitação de Certificado (Event ID 4886/4887) : No seu Domain Controller (HYDRA-DC), o Windows registra eventos de solicitação de certificado.

Event ID 4886: Uma nova solicitação de certificado foi recebida pela CA.

Event ID 4887: Uma solicitação de certificado foi emitida (aprovada).

Monitorar a Solicitação em Nome de Outro (Event ID 4888) :

Event ID 4888: Uma solicitação de certificado foi feita em nome de outro sujeito (o coração do ataque ESC2). Este evento é um indicador direto de que um agente de registro está sendo usado.
