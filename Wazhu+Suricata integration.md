- OPCIONAL: Baixamos e instalamos o Notepad++ -> https://notepad-plus-plus.org/downloads/
- Baixar e instalar o npcap -> https://npcap.com/#download
- Baixar e instalar o Suricata -> https://suricata.io/download/
- Baixar regras do Suricata -> https://rules.emergingthreats.net/open/
- Editar arquivo de configuração do Suricata.
- Editar: C:\Program Files\Suricata\Suricata.yaml
  
address-groups: HOME_NET: "IP_LOCAL"

rule-files:

emerging-all.rules (comentar o restante das regras)
e copiar e colar o arquivo baixado em suricata/rules

Iniciar o Suricata.
Obter o UUID (identificador único de hardware). wmic nicconfig get ipaddress,SettingID

IMPORTANTE: Nas novas versões do Windows, o WMIC está obsoleto. Vocês devem usar este comando:

Get-CimInstance Win32_NetworkAdapterConfiguration |
Where-Object { $_.IPAddress } |
Select-Object @{n="IPAddress";e={$_.IPAddress -join ", "}}, SettingID

Abrir o CMD como administrador e executar. "C:\Program Files\Suricata\suricata.exe" -c "C:\Program Files\Suricata\suricata.yaml" -i \Device\NPF_{2980F88B-DA35-459F-BD2B-01F61E384406}

Realizamos ping de outro computador e verificamos o eve.json.

Configurar o Wazuh-Agent.
Editar: C:\Program Files (x86)\ossec-agent\ossec.conf

<ossec_config> ... <log_format>json</log_format> C:\Program Files\Suricata\log\eve.json ... </ossec_config>

Reiniciar o serviço Wazuh-Agent -> CMD net stop Wazuh e net start Wazuh

Verificar o Wazuh Dashboard.


*************************************** CRIAR TAREFA AGENDADA ************************************************

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
