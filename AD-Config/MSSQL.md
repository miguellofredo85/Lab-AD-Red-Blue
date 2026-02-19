## 🔧 Passo 1: Abrir a Porta 60111 no Firewall do Windows

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

# Habilitar TCP/IP via registro (sem SQL Server Configuration Manager)
$regpath = "HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL*.MARVELLOCAL\MSSQLServer\SuperSocketNetLib\Tcp"
Set-ItemProperty -Path $regpath -Name "Enabled" -Value 1

# Configurar porta 60111
$regpath = "HKLM:\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL*.MARVELLOCAL\MSSQLServer\SuperSocketNetLib\Tcp\IPAll"
Set-ItemProperty -Path $regpath -Name "TcpPort" -Value "60111"
Set-ItemProperty -Path $regpath -Name "TcpDynamicPorts" -Value ""

# Criar login para SQLService
sqlcmd -S localhost\SQLEXPRESS -Q "
CREATE LOGIN [MARVEL\SQLService] FROM WINDOWS;
ALTER SERVER ROLE sysadmin ADD MEMBER [MARVEL\SQLService];
"

# Abrir firewall opcional
New-NetFirewallRule -DisplayName "SQL Server" -Direction Inbound -LocalPort 60111 -Protocol TCP -Action Allow

# Reiniciar serviço
Restart-Service -Name "MSSQL$SQLEXPRESS"

# Registrar SPNs para o SQLService
setspn -A MSSQLSvc/HYDRA-DC.MARVEL.local:60111 MARVEL\SQLService
```











