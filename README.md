# Laboratório Red/Blue Team com Active Directory, Wazuh, pfSense e Kali Linux

## 📋 Descrição do Projeto (em continuo desenvolvimento que vai ser incrementado aos poucos)

Este repositório documenta a implementação de um laboratório completo de segurança ofensiva (Red Team) e defensiva (Blue Team), utilizando uma infraestrutura baseada em Active Directory (AD) monitorada pelo Wazuh, com firewall pfSense e uma máquina Kali Linux para simulação de ataques.

O projeto tem como objetivo criar um ambiente controlado para estudo e prática de técnicas de ataque e defesa em um cenário corporativo simulado, permitindo a análise de detecção e resposta a incidentes.

## 🏗️ Arquitetura do Ambiente



               ┌─────────────────┐    ┌─────────────────┐
                  │ Kali Linux │          | Wazuh |
                  │ (Attack VM) │          | SIEM |
              └──────────────────┘    └─────────────────┘
              ┌──────────────────────────────────────┐
              │            REDE INTERNA              │
              │            (192.168.1.0/24)          │
              └──────────────────────────────────────┘
                               │
                  ┌────────────┴────────────┐
                  │         pfSense          │
                  │       (Firewall)         │
                  │      192.168.1.1         │
                  └────────────┬────────────┘
                               │
            ┌──────────────────|────────────────────┐
                               │                        
                               ▼                        
                      ┌─────────────────┐ 
                          │ Domain │                   
                        │ Controller │                
                        │ Windows Srv │       
                      └────────|─────────┘
                               |
                               ▼
      ┌────────────────────────┼────────────────────────┐
      │ 192.168.1.10 │ │ 192.168.1.100 │ │ 192.168.1.200 │
      └──────────────┘ └───────────────┘ └─--────-───────┘
            ├──────────────────┬──────────────────┐
            ▼                  ▼                  ▼
      ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
      │ Cliente 1 │           │ Cliente 2 │       │ Cliente 3 │
      │ Windows 10 │         │ Windows 10 │       │ Windows 10 │
      └─────────────────┘ └─────────────────┘ └─────────────────┘




## 🛠️ Componentes e Tecnologias

### 🔵 Blue Team (Defesa)
- **Active Directory (Windows Server)**: Controlador de domínio para simular ambiente corporativo
- **Wazuh**: SIEM/HIDS para coleta e análise de logs, monitoramento de integridade e detecção de ameaças
- **Sysmon**: Monitoramento avançado do Windows (logs de processo, conexões de rede, criação de arquivos)
- **VirusTotal**: Integração para análise de hashes de arquivos maliciosos
- **Suricata**: IDS/IPS para monitoramento de tráfego de rede e detecção de padrões de ataque
- **pfSense**: Firewall/UTM para controle de tráfego e segmentação de rede

### 🔴 Red Team (Ataque)
- **Kali Linux**: Distribuição para testes de penetração e simulação de ataques

## 🎯 Etapas de Configuração

### 1. Configuração Inicial da Infraestrutura
- Instalação e configuração do pfSense como gateway/firewall
- Configuração do Active Directory (Windows Server)
- Instalação e configuração do Wazuh (manager, indexer e dashboard)
- Integração do Wazuh em cada maquina windows

### 2. Configuração dos Agentes (Endpoint Windows)
- Instalação e configuração do Sysmon para coleta detalhada de eventos
- Configuração do agente Wazuh para coleta de logs do Sysmon
- Integração com VirusTotal para análise automatizada de hashes
- Aplicação de regras personalizadas de detecção

### 3. Configuração do IDS/IPS (Suricata)
- Instalação e configuração do Suricata
- Integração com Wazuh para envio de alertas
- Criação de regras personalizadas para detecção de tráfego malicioso

### 4. Configuração do Kali Linux (Máquina de Ataque)
- Preparação do ambiente de ataque
- Conexão à rede do laboratório
- Ferramentas instaladas: nmap, metasploit, crackmapexec, responder, bloodhound, mimikatz, etc.

## ⚔️ Cenários de Ataque e Configuração

Para cada ataque, este repositório documenta passo a passo:

### 1. **Reconhecimento e Enumeração**
   - **Ferramenta**: Nmap, BloodHound
   - **Configuração necessária**: Descoberta de hosts, serviços e coleta de informações do AD
   - **Comandos**: `nmap -sV 192.168.1.0/24`, `bloodhound-python -u user -p pass -d lab.local -dc dc.lab.local -c All`

### 2. **Ataques de Autenticação**
   - **Kerberoasting**
   - **AS-REP Roasting**
   - **Pass-the-Hash**
   - **SMB Relay**
   - **LLMNR/NBT-NS Poisoning** (Responder)
   -  **Delegacoes**
   -  **Certificados**

### 3. **Ataques de Senha**
   - **Password Spraying**
   - **Brute Force**
   - **Credential Dumping** (Mimikatz, secretsdump)

### 4. **Movimentação Lateral e Escalação de Privilégios**
   - **PsExec, WMI, WinRM**
   - **Token Impersonation**
   - **Unquoted Service Paths**
   - **Scheduled Tasks**

### 5. **Persistência**
   - **Criação de usuários locais/domínio**
   - **Golden Ticket**
   - **Silver Ticket**
   - **Diamond Ticket**


## 📊 Detecção e Resposta (Blue Team)

Para cada ataque documentado, são fornecidos:
- **Regras do Wazuh** para detecção
- **Consultas personalizadas** no dashboard do Wazuh
- **Procedimentos de resposta a incidentes**





## ⚠️ Aviso Legal

Este laboratório é destinado **APENAS para fins educacionais e de pesquisa em ambiente controlado**. Não utilize estas técnicas em sistemas sem autorização explícita. O autor não se responsabiliza pelo uso indevido das informações contidas neste repositório.
