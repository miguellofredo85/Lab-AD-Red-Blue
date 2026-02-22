## 🛠️ Configuração do AD para RBCD (Lab)

Por que permissão de escrita é necessário?
é obrigatório para o ataque RBCD porque o atacante precisa modificar o atributo msDS-AllowedToActOnBehalfOfOtherIdentity do computador alvo (SPIDERMAN$). Este atributo é uma lista de controle de acesso que define quais contas podem delegar para aquele computador .

Sem permissão de escrita neste atributo, não é possível configurar a delegação. As permissões que funcionam são :

Permissão	|Descrição|
|-|-|
GenericWrite	|Permissão geral de escrita no objeto|
GenericAll|	Controle total sobre o objeto|
WriteProperty	|Escrita em propriedades específicas|
WriteDacl	|Modificar a lista de controle de acesso|
WriteOwner	|Tomar posse do objeto|

- Para criar um ambiente vulnerável a RBCD, precisamos configurar permissões específicas no objeto SPIDERMAN$. Um atacante precisaria de uma conta com permissões de escrita no atributo msDS-AllowedToActOnBehalfOfOtherIdentity

- No DC, ir no Server Manager -> Tools -> Users and Computers
- Computer -> click direito no SPIDERMAN -> Properties -> Add -> fcastle (usuario) -> Write
  <img width="1535" height="766" alt="1" src="https://github.com/user-attachments/assets/6eadaa64-1e2a-43d4-8946-3278f0ecbb1f" />
- Advanced -> fcastle -> Edit -> check Write msDS-AllowedToActOnBehalfOfOtherIdentity -> Aplicar
<img width="1671" height="794" alt="2" src="https://github.com/user-attachments/assets/232b12e0-bce2-441d-bea7-e024e3b28724" />

- Com nxc verificamos se podemos adicionar maquinas, o maq tem que ser diferente de 0. Em este caso podemos adicionar 10.
```
  miguel~/Tools/ADplusWinTools nxc ldap 192.168.0.40 -u fcastle -p 'MYpassword#' -M maq
LDAP        192.168.0.40    389    HYDRA-DC         [*] Windows Server 2022 Build 20348 (name:HYDRA-DC) (domain:MARVEL.local) (signing:None) (channel binding:Never) 
LDAP        192.168.0.40    389    HYDRA-DC         [+] MARVEL.local\fcastle:MYpassword# 
MAQ         192.168.0.40    389    HYDRA-DC         [*] Getting the MachineAccountQuota
MAQ         192.168.0.40    389    HYDRA-DC         MachineAccountQuota: 10
miguel~/Tools/ADplusWinTools 
``` 

## Bloodhound 
- injestor
  ```miguel~ nxc ldap 192.168.0.40 -u fcastle -p 'MYpassword#' --bloodhound --collection All --dns-server 192.168.0.40```
- query
```
MATCH p=(u)-[:GenericWrite|GenericAll|WriteDacl|WriteOwner|Owns|WriteAccountRestrictions]->(c:Computer)
RETURN p
```

## Exploração desde Linux
<img width="1589" height="856" alt="GWfrom-linux-RCD" src="https://github.com/user-attachments/assets/aa3c56e1-df15-4865-9121-3c3d3235063f" />

- Adicionamos uma maquina
  <img width="1913" height="252" alt="addmachine" src="https://github.com/user-attachments/assets/7a237b1b-b4ee-45b6-b2bc-f2203305530c" />

- Agora comecamos o RBCD configurando a FAKECOMPUTER para delegar a SPIDERMAN
<img width="1065" height="663" alt="newblood" src="https://github.com/user-attachments/assets/fc015ecb-d345-4139-a2e4-184fd7cff1ca" />
- Adicionamos permisos
<img width="1296" height="202" alt="permisos" src="https://github.com/user-attachments/assets/194b1243-073c-42c6-9c24-d440c2eb4538" />


## Exploração desde Windows
<img width="1487" height="796" alt="blood-RBCD" src="https://github.com/user-attachments/assets/b43148a1-2157-435b-9b90-59effd15b031" />

## Proteções contra RBCD

## Detecção com Wazuh (Regras)
