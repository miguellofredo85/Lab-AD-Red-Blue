Existe um problema conhecido como “Double Hop” que surge quando um invasor tenta usar a autenticação Kerberos em dois (ou mais) saltos. O problema diz respeito à forma como os tickets Kerberos são concedidos para recursos específicos. Os tickets Kerberos não devem ser vistos como senhas. São dados assinados pelo KDC que indicam a quais recursos uma conta pode acessar. Quando realizamos a autenticação Kerberos, obtemos um “bilhete” que nos permite acessar o recurso solicitado (ou seja, uma única máquina). Por outro lado, quando usamos uma senha para autenticar, esse hash NTLM é armazenado em nossa sessão e pode ser usado em outro lugar sem problemas.
<img width="1229" height="386" alt="1" src="https://github.com/user-attachments/assets/64902250-5d96-4ebc-876d-f773f145c8cf" />


Soluções alternativas 

Algumas soluções alternativas para o problema do double-hop são abordadas nesta [publicação](https://posts.slayerlabs.com/double-hop/).

Solução alternativa nº 1: Objeto PSCredential

passando credenciais
```
*Evil-WinRM* PS C:\Users\backupadm\Documents> $SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force

|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK

*Evil-WinRM* PS C:\Users\backupadm\Documents>  $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)
*Evil-WinRM* PS C:\Users\backupadm\Documents> get-domainuser -spn -credential $Cred | select samaccountname
```

Solução alternativa nº 2: registrar a configuração da PSSession

Um truque que podemos usar aqui é registrar uma nova configuração de sessão usando o cmdlet [Register-PSSessionConfiguration](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-pssessionconfiguration?view=powershell-7.2).

Problema de “duplo salto” do Kerberos

```PS C:\> Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm```

Feito isso, precisamos reiniciar o serviço WinRM digitando `Restart-Service WinRM`
 em nossa PSSession atual. Isso nos desconectará, então iniciaremos uma nova 
PSSession usando a sessão registrada nomeada que configuramos anteriormente.

Depois de iniciar a sessão, podemos ver que o problema de salto duplo foi eliminado e, se digitarmos `klist`,
 teremos os tickets em cache necessários para acessar o controlador de domínio.
 Isso funciona porque nossa máquina local agora representará a máquina remota 
no contexto do usuário `backupadm` e
todas as solicitações da nossa máquina local serão enviadas diretamente para o controlador de domínio.

> Observação: não podemos usar Register-PSSessionConfiguration a partir de um
 shell winrm malicioso, pois não conseguiremos obter o pop-up de credenciais.
 Além disso, se tentarmos executar isso configurando primeiro um objeto PSCredential 
e, em seguida, tentando executar o comando passando credenciais 
como -RunAsCredential $Cred, receberemos uma mensagem de erro, pois só podemos usar RunAs
 a partir de um terminal PowerShell elevado. Portanto, esse método não 
funcionará por meio de uma sessão evil-winrm, pois requer acesso à GUI e um console PowerShell adequado.
 Além disso, em nossos testes, não conseguimos fazer esse 
método funcionar a partir do PowerShell em um host de ataque Parrot ou Ubuntu devido a 
certas limitações sobre como o PowerShell no Linux funciona com credenciais Kerberos
. Este método ainda é altamente eficaz se estivermos testando 
a partir de um host de ataque Windows e tivermos um conjunto de credenciais ou comprometermos um
 host e pudermos nos conectar via RDP para usá-lo como um “host de salto” para montar 
novos ataques contra hosts no ambiente. 

Também podemos usar outros métodos, como CredSSP, encaminhamento de porta ou injeção em um processo em execução no contexto de um usuário alvo (processo sacrificial).














