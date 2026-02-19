#  Abrir a Porta 60111 no Firewall do Windows

```
# Executar como ADMINISTRADOR no PowerShell

# Criar regra de entrada para a porta 60111 (TCP)
New-NetFirewallRule -DisplayName "SQL Server - Porta 60111" `
                    -Direction Inbound `
                    -LocalPort 60111 `
                    -Protocol TCP `
                    -Action Allow `
                    -Profile Domain,Private

# Verificar se a regra foi criada
Get-NetFirewallRule -DisplayName "SQL Server - Porta 60111" | Format-List
```

# No HYDRA-DC, como administrador
```
$url = "https://go.microsoft.com/fwlink/?linkid=2215158"  # SQL 2022 Express
$output = "C:\SQL2022-SSEI-Expr.exe"

Invoke-WebRequest -Uri $url -OutFile $output

# OU via BITS (mais estável para downloads grandes)
Start-BitsTransfer -Source $url -Destination $output

# Extrair arquivos
Start-Process -FilePath $output -ArgumentList "/ACTION=Download" -Wait

# Instalar (ajuste os parâmetros conforme necessário)
Start-Process -FilePath $output -ArgumentList @"
/ACTION=Install
/FEATURES=SQL
/INSTANCENAME=SQLEXPRESS
/TCPENABLED=1
/NPENABLED=0
/SECURITYMODE=SQL
/SAPWD="P@ssw0rd123!"
/SQLSYSADMINACCOUNTS="MARVEL\Administrator" "MARVEL\SQLService"
/QUIET
"@ -Wait

# Verificar instalação
Get-Service -Name "*SQL*"

# Caminho correto para SQL 2022
$regpath = "HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL16.MSSQLSERVER\MSSQLServer\SuperSocketNetLib\Tcp"

# Habilitar TCP/IP
Set-ItemProperty -Path $regpath -Name "Enabled" -Value 1

# Configurar porta 60111
$ipAllPath = "$regpath\IPAll"
Set-ItemProperty -Path $ipAllPath -Name "TcpPort" -Value "60111"
Set-ItemProperty -Path $ipAllPath -Name "TcpDynamicPorts" -Value ""

# Verificar configuração
Get-ItemProperty -Path $regpath | Select-Object Enabled
Get-ItemProperty -Path $ipAllPath | Select-Object TcpPort, TcpDynamicPorts

# Reiniciar serviço
Restart-Service -Name MSSQLSERVER -Force

# Aguardar alguns segundos
Start-Sleep -Seconds 10

# Verificar se porta está ouvindo (agora deve ser TCP)
netstat -an | findstr :60111

# Deve mostrar: TCP 0.0.0.0:60111 LISTENING


# Registrar SPNs para o SQLService
setspn -A MSSQLSvc/HYDRA-DC.MARVEL.local:60111 MARVEL\SQLService

```

# Criar login SEM sysadmin

```
SQLCMD.exe -S localhost -E -Q @"
-- Criar login (se não existir)
IF NOT EXISTS (SELECT * FROM sys.server_principals WHERE name = 'MARVEL\SQLService')
    CREATE LOGIN [MARVEL\SQLService] FROM WINDOWS;
GO

-- Remover qualquer permissão de sysadmin (garantir que não tem)
ALTER SERVER ROLE sysadmin DROP MEMBER [MARVEL\SQLService];
GO

-- Criar banco de dados para teste
IF NOT EXISTS (SELECT * FROM sys.databases WHERE name = 'TestDB')
    CREATE DATABASE TestDB;
GO

-- Dar acesso apenas ao TestDB
USE TestDB;
GO
IF NOT EXISTS (SELECT * FROM sys.database_principals WHERE name = 'SQLService')
    CREATE USER [SQLService] FOR LOGIN [MARVEL\SQLService];
GO

-- Dar apenas permissão de leitura (db_datareader)
ALTER ROLE db_datareader ADD MEMBER [SQLService];
GO

-- Verificar permissões
SELECT 'Login criado:' as Info, name 
FROM sys.server_principals 
WHERE name = 'MARVEL\SQLService';
GO

USE TestDB;
GO
SELECT 'Usuário no banco:' as Info, name, 
       (SELECT name FROM sys.database_role_members rm 
        JOIN sys.database_principals r ON rm.role_principal_id = r.principal_id
        WHERE rm.member_principal_id = dp.principal_id) as Role
FROM sys.database_principals dp
WHERE dp.name = 'SQLService';
GO
"@
```
# 🔧 Verificar permissões (NÃO deve ser sysadmin)
```
& $sqlcmd -S localhost -E -Q @"
-- Verificar se NÃO é sysadmin
SELECT 
    'MARVEL\SQLService' as Login,
    CASE 
        WHEN IS_SRVROLEMEMBER('sysadmin', 'MARVEL\SQLService') = 1 
        THEN '❌ TEM SYSMADMIN (ERRO!)' 
        ELSE '✅ NÃO tem sysadmin (CORRETO)' 
    END as StatusSysadmin;
GO

-- Testar acesso ao banco (deve funcionar)
USE TestDB;
GO
SELECT 'Acesso ao TestDB: OK' as Teste;
GO

-- Tentar acessar master (NÃO deve funcionar)
USE master;
GO
SELECT 'Se você ver isso, tem permissão demais!' as Aviso;
GO
"@
```










