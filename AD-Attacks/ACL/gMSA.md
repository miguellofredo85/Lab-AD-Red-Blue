- [Introdução](#introdução)
- [Configuração](#configuração) 
- [Ataque](#ataque)
- [Prevenção](#prevenção)
- [Detecção](#detecção)



## Introdução
Para configurar e explorar a permissão ReadGMSAPassword em seu laboratório, é importante primeiro entender o que é uma gMSA (Group Managed Service Account). Essas contas são especiais porque o Windows gerencia sua senha automaticamente (de 240 caracteres) e a altera a cada 30 dias.
O ataque se baseia no fato de que a permissão de leitura dessa senha geralmente não é restrita apenas à conta que usa o serviço, mas às vezes é delegada por engano a usuários ou grupos com menos privilégios.

## Configuração
> Adicionar gupo gMSA
> No Server Manager -> Tools -> Users and Computers -> Pasta Groups click direito -> New -> Group -> coloca o nome
<img width="1222" height="531" alt="gmsa_group" src="https://github.com/user-attachments/assets/52c8474c-0a1f-42f7-90b6-d7bc3d986d1f" />

> Adicionar HYDRA-DC$ (computer) e usuario ckent no gmsa_group
> No Server Manager -> Tools -> Users and Computers -> Pasta do User ou Computers -> procure o usuario ou computador e click direito
<img width="1677" height="520" alt="add-computer-to-group-gmsa" src="https://github.com/user-attachments/assets/5a54b162-aabb-475c-b04f-99b76833cf6c" />
<img width="1643" height="541" alt="add-user-to-group-gmsa" src="https://github.com/user-attachments/assets/3f9d5cb7-70ad-466b-9f68-498f8dbd7eb4" />

> Criamos a KDS Root Key no Dominio e fazemos o test com ``` Get-KdsRootKey ``` 
<img width="818" height="399" alt="kds-root-key" src="https://github.com/user-attachments/assets/2743d129-f81d-40e0-8f1e-e128674bef37" />

> Criamos a conta gMSA e fazemos o test
- ```New-ADServiceAccount -Name "MyGMSA" -DNSHostName "dc.ignite.local" -PrincipalsAllowedToRetrieveManagedPassword "gmsa_group"```
- ```Get-ADServiceAccount MyGMSA -Properties PrincipalsAllowedToRetrieveManagedPassword```
<img width="1229" height="291" alt="test-service-account" src="https://github.com/user-attachments/assets/df3b92c8-8d4f-400b-acd1-a9d2d8efff5e" />
<img width="1152" height="365" alt="gui-verify-service-account" src="https://github.com/user-attachments/assets/af5a13f1-3d07-474a-b2f1-1517473f856a" />

> Atribuir um SPN:
> Para permitir a autenticação via Kerberos, atribua um Nome Principal do Serviço (SPN)
> Este comando registra um Nome Principal do Serviço (SPN) para a conta gMSA MyGMSA no domínio ignite.local. Ele permite que a conta seja autenticada usando Kerberos para o serviço especificado
```setspn -a MSSQLSvc/HYDRA-DC.MARVEL.local MARVEL.local\MyGMSA```
<img width="1075" height="609" alt="set-spn" src="https://github.com/user-attachments/assets/dfae79d5-efad-4c9c-9fe6-5006ebb9fa4a" />

Parâmetro | 	O que significa |	Explicação no seu contexto|
|-|-|
setspn |	Ferramenta de Comando	| Utilitário nativo do Windows para gerenciar os nomes de serviço.|
-a | 	Add (Adicionar)	| Instrução para adicionar um novo registro de SPN à conta.|
MSSQLSvc/	| Service Class	Define o tipo de serviço. | MSSQLSvc é o padrão para SQL Server.|
HYDRA-DC.MARVEL.local	Host/Port	| Onde o serviço está rodando. | Aqui você está dizendo que o serviço SQL está no seu Domain Controller.|
MARVEL.local\MyGMSA	| Target Account	A conta que "será dona" desse serviço. | É aqui que você vincula o privilégio à conta gMSA.|



> Instalacao de gMSA no HYDRA-DC
> ```Add-WindowsCapability -Online -Name Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0```

> O comando abaixo instala a conta gMSA (MyGMSA) na máquina local, permitindo que ela recupere e use a senha gerenciada. A máquina deve fazer parte do grupo de segurança associado ao gMSA, ou a instalação falhará. Atribuir um SPN:
```Install-ADServiceAccount -Identity "MyGMSA"```
```Test-ADServiceAccount -Identity "MyGMSA"```


> Conta gMSA (MyGMSA) na máquina local, permitindo que ela recupere e use a senha gerenciada. A máquina deve fazer parte do grupo de segurança associado ao gMSA, ou a instalação falhará.
``` Install-ADServiceAccount -Identity MyGMSA ```
> Se falhar, incluimos o computado como membro do grupo gmsa, logo limpamos a cache do kerberos do equipo, forcamos atualizacao das policas e instalamos
<img width="1359" height="498" alt="falha1" src="https://github.com/user-attachments/assets/11d493b4-d319-4570-86f2-8249482a5272" />
<img width="839" height="368" alt="install" src="https://github.com/user-attachments/assets/1e9132d2-d313-41a3-835f-51f9d5cc9e97" />
<img width="745" height="48" alt="test" src="https://github.com/user-attachments/assets/81f37937-255e-4a80-9995-4ec88ec81194" />



## Ataque
> Reconhecimento com netexect obtemos o .zip para injestar no bloodhound
<img width="1665" height="134" alt="blood1" src="https://github.com/user-attachments/assets/3f7da5ba-fd19-478f-b2ba-7daad653b249" />
<img width="1823" height="850" alt="blood2" src="https://github.com/user-attachments/assets/50cd71bd-d531-4509-837f-4446dee0a4a3" />

> Accao
> Baixamos a ferramenta
> ```git clone https://github.com/micahvandeusen/gMSADumper.git```
> ``` pip install -r requirements.txt  --break-system-packages```
<img width="981" height="140" alt="hashGMSA" src="https://github.com/user-attachments/assets/f9471ad5-de64-4a4d-87d4-cb9aa551cdfe" />

> Ou com netexec
<img width="1749" height="118" alt="nxchashgmsa" src="https://github.com/user-attachments/assets/becf3c94-9f96-495d-8b12-645eb0931007" />

> Log on winrm, de aqui para a frente voce pode procurar para fazer pivoting ou outras ACL e etc
<img width="1099" height="232" alt="winrm" src="https://github.com/user-attachments/assets/76fe2e29-dbdd-40ff-b7a4-24b56035bfbe" />



## Prevenção
- Aplique o princípio do privilégio mínimo: audite e restrinja regularmente as permissões para modificar contas gMSA, garantindo que apenas as entidades necessárias tenham acesso.
- Monitore o acesso ao gMSA: revise continuamente o atributo msDS-GroupMSAMembership para garantir que apenas contas de computador autorizadas possam recuperar a senha.
- Habilite alertas em tempo real: configure o monitoramento para detectar e alertar sobre quaisquer alterações nas permissões do gMSA, permitindo uma resposta rápida a possíveis ameaças.


## Detecção
> Wazuh + Sysmon (Logs de Eventos)
> O Windows gera o Event ID 4662 quando um objeto do AD é acessado. Precisamos filtrar pelo GUID do atributo da senha da gMSA.
```
<group name="windows,active_directory,">
  <rule id="110010" level="12">
    <if_sid>60103</if_sid>
    <field name="win.system.eventID">^4662$</field>
    <field name="win.eventdata.properties">3d1a588a-367d-4114-8789-f5383f98018e</field>
    <description>Alerta: Leitura da senha de uma conta gMSA detectada (Possível escalação lateral)</description>
  </rule>
</group>
```
