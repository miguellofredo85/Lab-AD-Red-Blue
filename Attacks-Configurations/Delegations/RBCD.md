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

## Exploração desde Linux

## Exploração desde Windows

## Proteções contra RBCD

## Detecção com Wazuh (Regras)
