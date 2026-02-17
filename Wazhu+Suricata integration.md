




***************************************************************************************************************
********************************************** SURICATA *******************************************************
***************************************************************************************************************

0. OPCIONAL: Descargamos e instalamos Notepad++ -> https://notepad-plus-plus.org/downloads/
1. Descargar e instalar npcap -> https://npcap.com/#download
2. Descargar e instalar Suricata -> https://suricata.io/download/
3. Descargar reglas Suricata -> https://rules.emergingthreats.net/open/
4. Editar archivo de configuración de Suricata.

Editar: C:\Program Files\Suricata\Suricata.yaml
  
address-groups:
    HOME_NET: "IP_LOCAL"

rule-files:
 - emerging-all.rules (comentar el resto de reglas)
 
 y copiar y pegar el archivo descargado en suricata/rules

5. Iniciar Suricata.

Obtener UUID (identificador único de hardware).
wmic nicconfig get ipaddress,SettingID

IMPORTANTE: En las nuevas versiones de Windows WMIC está deprecado. Debéis emplear este comando:

Get-CimInstance Win32_NetworkAdapterConfiguration |
Where-Object { $_.IPAddress } |
Select-Object @{n="IPAddress";e={$_.IPAddress -join ", "}}, SettingID

Abrir CMD como administrador y ejecutar.
"C:\Program Files\Suricata\suricata.exe" -c "C:\Program Files\Suricata\suricata.yaml" -i \Device\NPF_{2980F88B-DA35-459F-BD2B-01F61E384406}

Realizamos ping desde otro equipo y comprobamos eve.json.

6. Configurar Wazuh-Agent.

Editar: C:\Program Files (x86)\ossec-agent\ossec.conf

<ossec_config>
...
  <localfile>
    <log_format>json</log_format>
    <location>C:\Program Files\Suricata\log\eve.json</location>
  </localfile>
...
</ossec_config>

Reiniciar servicio Wazuh-Agent -> CMD
net stop Wazuh
net start Wazuh

7. Comprobar Wazuh Dashboard.

***************************************************************************************************************
*************************************** CREAR TAREA PROGRAMADA ************************************************
***************************************************************************************************************

1. Abre el Programador de tareas (Task Scheduler) desde el menú de inicio.

2. Haz clic en Crear tarea básica.

3. Ponle un nombre (por ejemplo, "Suricata IDS").

4. Elige cuando se inicie el sistema como desencadenador.

5. En Acción, selecciona Iniciar un programa.

    - En Programa o script, pon la ruta al ejecutable de Suricata:

        "C:\Program Files\Suricata\suricata.exe"

    - En Agregar argumentos, pon:

        -c "C:\Program Files\Suricata\suricata.yaml" -i \Device\NPF_{*************UUID*************}

6. Ejecuta la tarea.
Mostrando WZ.06 SURICATA WINDOWS.txt.
