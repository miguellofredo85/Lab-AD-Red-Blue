👉 [Explicação](#explicação)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

## Explicação
O Kerberos Golden Ticket é um ataque no qual agentes maliciosos podem criar/gerar tickets para qualquer usuário no domínio, atuando efetivamente como um controlador de domínio.

> Este ataque fornece meios para uma persistência elevada no domínio. Ocorre depois que um adversário obtém privilégios de administrador de domínio (ou similares)

Imagina que a tua empresa tem um cartão de acesso que funciona para todas as portas. Quem controla a emissão desses cartões é uma entidade central (o Controlador de Domínio).
No centro deste sistema, existe uma conta de computador especial, chamada krbtgt. Pensa nela como o "carimbo oficial" da empresa. Para um cartão ser válido, tem de ter este carimbo. É a chave-mestre que garante que todos os cartões são legítimos.
O Ataque do Golden Ticket (Bilhete Dourado) acontece quando um atacante consegue roubar uma cópia desse "carimbo oficial" (que é o hash da password da conta krbtgt).
A partir desse momento, o atacante pode fabricar os seus próprios cartões de acesso em qualquer computador, sem precisar de autorização do sistema central. Como os cartões falsos têm o carimbo oficial, o sistema abre todas as portas, achando que são legítimos.
Com este ataque, o criminoso pode:
- Criar um cartão que lhe dá acesso a todas as máquinas e serviços da empresa.
- Fazer-se passar por qualquer pessoa, incluindo o administrador principal.
- Manter o acesso mesmo que a password do administrador seja mudada, porque ele tem o controlo do carimbo que valida tudo.
- Resumo simples: É o ataque mais poderoso num domínio do Windows. Quem tem o "carimbo" (hash do krbtgt) tem o poder de criar o seu próprio "passe dourado" para toda a rede.

## Ataque

- Pegamos o hash do krbtgt
``` mimikatz # lsadump::dcsync /domain:MARVEL.local /user:krbtgt```
Onde:
/domain: O nome do domínio.
/sid: O valor SID do domínio.
/rc4: O hash da senha do krbtgt.
/user: O nome de usuário para o qual o Mimikatz emitirá o ticket (o Windows 2019 bloqueia tickets se forem para usuários inexistentes).
/id: ID relativo (última parte do SID) do usuário para o qual o Mimikatz emitirá o ticket.

- SID do dominio (importar PowerView.ps1)
```
PS C:\Users\Administrator\Downloads> Get-DomainSID
S-1-5-21-2699579203-2312383683-555258765
```
- Forjamos o ticket do administrator e mediante /ptt (pass the ticket) salvamos o ticket dentro da sessao atual que pudemos ver com klis
<img width="724" height="448" alt="GT1" src="https://github.com/user-attachments/assets/40293def-aa29-4046-986a-82bec6fec28e" />

<img width="1714" height="358" alt="GT2" src="https://github.com/user-attachments/assets/ca016cf9-a663-44ac-912d-d9a99be632c1" />


- Ticket salvado 
<img width="716" height="346" alt="KLIST" src="https://github.com/user-attachments/assets/bd816058-2a7b-4700-91eb-c794c4696d72" />


- Agora podemos listar
<img width="674" height="387" alt="List" src="https://github.com/user-attachments/assets/1901e455-ee40-4045-8ea9-1f1cc4ad7ac5" />



## Prevenção

- Protege o "Carimbo" (krbtgt): Muda a password da conta krbtgt regularmente (de 6 em 6 meses ou anualmente). Usa o script oficial da Microsoft (KrbtgtKeys.ps1) para o fazer em modo de auditoria primeiro, garantindo que todos os Controladores de Domínio sincronizam sem causar problemas na rede.

- Protege os Administradores: Bloqueia que utilizadores com privilégios (como Administradores de Domínio) iniciem sessão em computadores normais ou servidores menos críticos. Assim, se um computador normal for infetado, o hash do krbtgt não estará lá para ser roubado.

- Protege os Limites (Filtragem SIDHistory): Ativa a filtragem de SIDHistory entre domínios (pai/filho). Isto impede que um atacante que domine um domínio filho consiga saltar para o domínio pai (o principal), fechando essa porta de escalada de privilégios.


## Detecção

<img width="1886" height="169" alt="wa" src="https://github.com/user-attachments/assets/1a73f03b-704f-4580-a876-14d52d1f5b59" />


```
<rule id="110017" level="12">
  <if_sid>60103</if_sid>  <!-- Grupo de eventos de segurança do Windows -->
  <field name="win.system.eventID">^4662$</field>
  <field name="win.eventdata.properties" type="pcre2">{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}|{19195a5b-6da0-11d0-afd3-00c04fd930c9}</field>
  <field name="win.eventdata.SubjectUserName" negate="yes">\$$</field> <!-- Ignora contas de computador (que terminam com $) -->
  <description>🚨 Possível ataque DCSync detectado - Conta: $(win.eventdata.SubjectUserName) tentou replicação de diretório</description>
  <mitre>
    <id>T1003.006</id> <!-- ATT&CK para DCSync / Credential Dumping -->
  </mitre>
</rule>
```







