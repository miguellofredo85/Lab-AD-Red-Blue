- OPCIONAL: Baixamos e instalamos o Notepad++ -> https://notepad-plus-plus.org/downloads/
- Baixar e instalar o npcap -> https://npcap.com/#download
- Baixar e instalar o Suricata -> https://suricata.io/download/
- Baixar regras do Suricata -> https://rules.emergingthreats.net/open/emerging-all.rules.zip

### Editar arquivo de configuração do Suricata.
- Editar: C:\Program Files\Suricata\Suricata.yaml
  
address-groups: HOME_NET: "IP_LOCAL"
<img width="999" height="753" alt="ip home net" src="https://github.com/user-attachments/assets/a2d5e48b-0ee1-4cf6-9147-7a72e1784a5e" />


rule-files:
copiar e colar o arquivo baixado em suricata/rules
<img width="1169" height="754" alt="commentrules" src="https://github.com/user-attachments/assets/fed9f73f-67fa-478c-94a2-4ad6169c8cc2" />

emerging-all.rules (comentar o restante das regras)

<img width="1169" height="754" alt="commentrules" src="https://github.com/user-attachments/assets/6074e8bf-eb2f-4d83-88f5-c821e725df58" />

Iniciar o Suricata.
Obter o UUID (identificador único de hardware). wmic nicconfig get ipaddress,SettingID

IMPORTANTE: Nas novas versões do Windows, o WMIC está obsoleto. Vocês devem usar este comando:

```Get-CimInstance Win32_NetworkAdapterConfiguration |```
```Where-Object { $_.IPAddress } |```
```Select-Object @{n="IPAddress";e={$_.IPAddress -join ", "}}, SettingID```

Abrir o CMD como administrador e executar. "C:\Program Files\Suricata\suricata.exe" -c "C:\Program Files\Suricata\suricata.yaml" -i \Device\NPF_{UUID}

<img width="1736" height="744" alt="runsuri" src="https://github.com/user-attachments/assets/ee4885f2-d70b-44d8-89cf-61dafc02d460" />

Realizamos ping de outro computador e verificamos o eve.json e a criacao dos arquivos.
<img width="1354" height="719" alt="log" src="https://github.com/user-attachments/assets/56fbfc42-b22e-4820-8c42-46cd6242349c" />


### Configurar o Wazuh-Agent.
Editar: C:\Program Files (x86)\ossec-agent\ossec.conf

```<ossec_config> <localfile> <log_format>json</log_format> <location>C:\Program Files\Suricata\log\eve.json</location> </localfile></ossec_config> </ossec_config>```

Reiniciar o serviço Wazuh-Agent -> CMD 
net stop Wazuh
net start Wazuh

Verificar o Wazuh Dashboard.


*************************************** CRIAR TAREFA AGENDADA PARA COMECAR SURICATA NO LOGIN************************************************

Abra o Agendador de tarefas (Task Scheduler) no menu iniciar.

Clique em Criar tarefa básica.

Dê um nome (por exemplo, "Suricata IDS").

Escolha quando o sistema iniciar como gatilho.

Em Ação, selecione Iniciar um programa.

Em Programa/script, coloque o caminho para o executável do Suricata:

"C:\Program Files\Suricata\suricata.exe"

Em Adicionar argumentos, coloque:

-c "C:\Program Files\Suricata\suricata.yaml" -i \Device\NPF_{UUID}

Execute a tarefa. Mostrando WZ.06 SURICATA WINDOWS.txt.
