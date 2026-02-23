- Otorgar SPN no usuario
  <img width="1411" height="427" alt="spn" src="https://github.com/user-attachments/assets/bb740067-0721-406d-98e2-17c9bd92ce41" />

- No DC (PowerShell como Administrador)

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

### No Windows
<img width="1430" height="139" alt="spn-service-onwindows" src="https://github.com/user-attachments/assets/a062547e-9357-4956-a0b4-7326543c6e10" />

Kerberoasting attack from linux
- como temos as pass de um usuario podemos kerberostear outros usuarios
<img width="1242" height="447" alt="ker-attack-linux" src="https://github.com/user-attachments/assets/6846b2c8-a8ee-415d-8ec1-6f75d926e6a4" />

Kerberoasting attack from Windows
.\Rubeus.exe kerberoast /outfile:spn.txt
<img width="1055" height="622" alt="ww" src="https://github.com/user-attachments/assets/e2dfd805-fc0c-4faa-82c0-577899cf6692" />


## Prevencao
- força da senha da conta de serviço
- limitar o número de contas com SPNs e desativar aquelas que não são mais utilizadas/necessárias

## Wazuh log e rule
```
<group name="security_event,windows">
    <!-- KERBEROASTING (T1558.003) -->
    <rule id="110002" level="12">
        <if_sid>60103</if_sid>
        <field name="win.system.eventID">^4769$</field>
        <!-- Ticket Encryption Type RC4 (0x17) = Kerberoasting -->
        <field name="win.eventdata.TicketEncryptionType" type="pcre2">0x17</field>
        <!-- Target NAO debe terminar en $ (nao e conta de máquina) -->
        <field name="win.eventdata.TargetUserName" type="pcre2">^[^$]+$</field>
        <!-- TicketOptions flexible para Rubeus/Impacket -->
        <field name="win.eventdata.TicketOptions" type="pcre2">0x40810000|0x40810010</field>
        <options>no_full_log</options>
        <description>Possible Kerberoasting attack from $(win.eventdata.IpAddress) for service $(win.eventdata.ServiceName)</description>
        <mitre>
            <id>T1558.003</id>
        </mitre>
    </rule>
</group>
```

<img width="1917" height="928" alt="Wazuh-Kerberoasting" src="https://github.com/user-attachments/assets/903096a4-1901-425d-990e-bd905008b7d6" />

  
