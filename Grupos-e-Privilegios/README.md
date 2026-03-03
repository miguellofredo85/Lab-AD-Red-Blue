# Adicionar Usuário a um Grupo
Via Interface Gráfica (GUI)
Standalone: Pressione Win + R, digite lusrmgr.msc. Vá em Grupos, clique duas vezes no grupo desejado e clique em Adicionar.

Active Directory: No Servidor, abra Usuários e Computadores do Active Directory (dsa.msc). Localize o usuário, clique com o botão direito > Adicionar a um grupo....

Via PowerShell
Para adicionar o usuário "Joao" ao grupo "Administradores":

Standalone:

`Add-LocalGroupMember -Group "Administradores" -Member "Joao"`

Active Directory:

`Add-ADGroupMember -Identity "Administradores do Dominio" -Members "Joao"`

---

# Atribuir o Privilégio 

1. Via Interface Gráfica (GUI)
O console central para isso é o Editor de Política de Segurança Local (em máquinas isoladas) ou o Gerenciamento de Política de Grupo (no AD).

Em Windows Standalone (Local)
Pressione Win + R, digite secpol.msc e dê Enter.

Navegue até: Configurações de Segurança > Políticas Locais > Atribuição de Direitos de Usuário.

No painel direito, você verá a lista completa (ex: Permitir logon local, Gerar auditorias de segurança, Fazer backup de arquivos).

Clique duas vezes no privilégio e adicione o Usuário ou Grupo.

Em Active Directory (Domínio)
No AD, você não configura máquina por máquina; você usa uma GPO:

No Controlador de Domínio, abra o gpmc.msc.

Edite uma GPO existente ou crie uma nova vinculada à Unidade Organizacional (OU) onde estão os computadores.

O caminho é o mesmo: Configuração do Computador > Políticas > Configurações do Windows > Configurações de Segurança > Políticas Locais > Atribuição de Direitos de Usuário.

2. Via PowerShell (O jeito "Ninja")
Como o PowerShell não tem um comando nativo simples (como um Set-Privilege), os administradores usam duas abordagens principais:

Opção A: Usando o módulo UserPrivileges (Recomendado)
Existe um módulo da comunidade muito popular que simplifica isso drasticamente.

PowerShell
```
# Instalar o módulo (requer internet e admin)
Install-Module -Name UserPrivileges

# Adicionar o privilégio de "Logon como Serviço" para um usuário
Add-UserPrivilege -Identity "NOME-DO-USUARIO" -Privilege SeServiceLogonRight
```
Opção B: Usando a ferramenta Secedit (Nativa)
Se você não puder instalar módulos, terá que usar o secedit para exportar a base de dados de segurança, editá-la e importá-la novamente. É um processo mais bruto:

Exportar: `secedit /export /cfg C:\temp\cfg.inf`

Editar: Abrir o .inf e adicionar o SID do usuário na linha do privilégio correspondente (ex: SeBackupPrivilege).

Importar: `secedit /configure /db $env:windir\security\local.sdb /cfg C:\temp\cfg.inf /overwrite`

---

## 🛡️ Vetores de Escalação de Privilégios (Windows)

Esta tabela resume os grupos e privilégios que frequentemente permitem a elevação de um usuário comum para **SYSTEM** ou **Administrator**.

### 👥 Grupos de Alto Impacto
| Grupo | Impacto | Descrição do Abuso |
| :--- | :--- | :--- |
| **Administradores** | Crítico | Controle total sobre a máquina local. No AD, permite extrair hashes do NTDS.dit. |
| **Backup Operators** | Alto | Permite ler qualquer arquivo no sistema (incluindo SAM e SYSTEM), ignorando ACLs. |
| **Server Operators** | Alto | Pode modificar serviços, mudar status de drivers e fazer backup/restauração. |
| **Remote Management Users** | Médio/Alto | Permite acesso via WinRM (PowerShell Remoto), facilitando movimentação lateral. |
| **DnsAdmins** | Crítico (AD) | Permite carregar uma DLL arbitrária com privilégios de SYSTEM no serviço de DNS. |
| **Account Operators** | Médio (AD) | Pode modificar propriedades e resetar senhas de contas (exceto outros Admins). |
| **Hyper-V Administrators** | Alto | Controle total sobre VMs, permitindo acesso aos discos rígidos virtuais dos DCs. |

### 🔑 Privilégios de Usuário (User Rights Assignment)
| Privilégio (Nome Técnico) | Nome na Interface (GUI) | Potencial de Abuso |
| :--- | :--- | :--- |
| **SeImpersonatePrivilege** | Representar um cliente após a autenticação | Uso de ferramentas como *Juicy/PrintSpoofer* para virar SYSTEM. |
| **SeBackupPrivilege** | Fazer backup de arquivos e diretórios | Permite ler a colmeia SAM/SYSTEM para extrair senhas locais. |
| **SeRestorePrivilege** | Restaurar arquivos e diretórios | Permite substituir binários de serviços por versões maliciosas (Backdoor). |
| **SeTakeOwnershipPrivilege** | Tornar-se proprietário de objetos | Permite assumir posse de arquivos críticos (ex: `sethc.exe`) e alterar ACLs. |
| **SeDebugPrivilege** | Depurar programas | Permite injetar código no `lsass.exe` para extrair credenciais em texto claro. |
| **SeLoadDriverPrivilege** | Carregar/Descarregar drivers | Permite instalar drivers vulneráveis para executar código em nível de Kernel. |
| **SeTcbPrivilege** | Agir como parte do SO | Permite criar tokens de segurança e se passar por qualquer usuário. |
| **SeServiceLogonRight** | Logar como um serviço | Frequentemente abusado se a conta tiver permissões de escrita no binário do serviço. |

> **⚠️ Aviso:** Esta tabela é para fins de auditoria e testes de penetração autorizados. O uso indevido é ilegal.












