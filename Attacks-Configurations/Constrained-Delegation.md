## Configuracao com Protocol Transition 
<img width="936" height="848" alt="" src="https://github.com/user-attachments/assets/577b7c01-6fc9-4416-bbea-91a933d1a683" />

## Configuracao sem Protocol Transition 
  <img width="955" height="857" alt="" src="https://github.com/user-attachments/assets/0ce8f95f-b4b0-4e97-b685-1b81cbe98080" />


## Com Protocol Transition PoC
<img width="1918" height="620" alt="poc-impacket" src="https://github.com/user-attachments/assets/58001aa8-2ead-4620-8697-b6df5cc215be" />

## Sem Protocol Transition 
Depois de desativar a Transição de Protocolo, o ataque direto falhará. A razão é que, sem essa opção, o ticket que fcastle obtém para si mesmo em nome de outro usuário (via S4U2Self) não é “encaminhável”. Como não pode ser reencaminhado, a etapa S4U2Proxy não funciona. É como se o DC dissesse: “Sei que você diz que o Administrador se conectou a você, mas como não foi com Kerberos, não vou deixar você usar essa identidade para se conectar a outro lugar”.
<img width="1917" height="243" alt="noforward" src="https://github.com/user-attachments/assets/e288d731-0cae-4850-9bfa-75cd0013531f" />
Você precisa de uma estratégia diferente. A mais comum é usar um ataque de Delegação Baseada em Recursos Restritos (Resource-Based Constrained Delegation).



