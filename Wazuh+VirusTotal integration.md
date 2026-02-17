1. Edita el archivo de configuración del agente Wazuh
C:\Program Files (x86)\ossec-agent\ossec.conf

<syscheck>
...
    <directories check_all="yes" report_changes="yes" realtime="yes">C:\Users\clockworker\Downloads</directories>
...
</syscheck>

2. Reinicia Wazuh-Agent - CMD con permisos de administrador
net stop Wazuh
net start Wazuh


Registro en VIRUSTOTAL -> https://www.virustotal.com/

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

3. Crear archivo eicar.txt en directorio monitorizado.

  X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*

4. Visualizar alerta en Wazuh Dashboard

5. Crear archivo malwaretest.txt  
 
  malwaretest
Mostrando WZ.07 - FIM & VIRUSTOTAL.txt.
