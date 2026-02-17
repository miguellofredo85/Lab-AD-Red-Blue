1. Edita arquivo de configuracaon do agente Wazuh
C:\Program Files (x86)\ossec-agent\ossec.conf

<syscheck>
...
    <directories check_all="yes" report_changes="yes" realtime="yes">C:\Users\clockworker\Downloads</directories>
...
</syscheck>

2. Reinicia Wazuh-Agent - CMD com permisos de administrador
net stop Wazuh
net start Wazuh


Registro no VIRUSTOTAL -> https://www.virustotal.com/

1. Configurar WAZUH-MANAGER

EDITAR: sudo nano /var/ossec/etc/ossec.conf

<integration>
  <name>virustotal</name>
  <api_key>0509d28e8d0ebb41dd11.......</api_key>
  <group>syscheck</group>
  <alert_format>json</alert_format>
</integration>

2. Reiniciar WAZUH-MANAGER

  systemctl restart wazuh-manager

3. Criar arquivo eicar.txt no directorio monitorizado.

X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*
<img width="1537" height="349" alt="eicar-test-virus" src="https://github.com/user-attachments/assets/2cbebe0f-849c-43d0-8ebf-dcba7d3f38f7" />


4. Visualizar alerta en Wazuh Dashboard
<img width="1897" height="593" alt="wazuh-virutotal" src="https://github.com/user-attachments/assets/ce5c1287-cf22-4730-9cc0-82a5dd24aeb5" />

