1️⃣ Configurar pfSense para enviar logs a Wazuh:

En la GUI de pfSense:

1. Ve a:
    **Status → System Logs → Settings**
2. Marca **Enable Remote Logging**.
3. En **Remote log servers** pon:
    192.168.30.2:514    
    (IP del Wazuh Manager y puerto 514/UDP)
4. En **Remote Syslog Contents** marcamos todo, para pruebas.
5. Guarda y aplica.

2️⃣ Configurar Wazuh Manager para recibir syslog

En el manager, edita:

    sudo nano /var/ossec/etc/ossec.conf

Dentro de `<ossec_config>`, añade (o ajusta) **estos bloques**:

🔹 Bloque para agentes (está opr defecto):

<remote>
  <connection>secure</connection>
  <port>1514</port>
  <protocol>tcp</protocol>
  <queue_size>131072</queue_size>
</remote>


🔹 Bloque nuevo para SYSLOG de pfSense:

<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>udp</protocol>
  <allowed-ips>192.168.30.1/32</allowed-ips>  <!-- IP de pfSense -->
</remote>

*(Puedes usar una red, por ejemplo `192.168.30.0/24` si quieres más dispositivos.)*

🔹 Activar que Wazuh guarde todos los logs (archives)

En la misma `ossec.conf`, dentro de `<global>`:

<global>
  <jsonout_output>yes</jsonout_output>
  <alerts_log>yes</alerts_log>
  <logall>yes</logall>
  <logall_json>yes</logall_json>
</global>

Guarda el archivo.

Reinicia el manager:
    sudo systemctl restart wazuh-manager
    sudo systemctl status wazuh-manager

Debe quedar en `active (running)`.

3️⃣ Comprobar que Wazuh está escuchando y recibiendo

🔍 Escuchando en 514/UDP
    ss -tulnp | grep 514

Tienes que ver algo tipo:
udp UNCONN 0 0 192.168.30.2:514 0.0.0.0:* users:(("wazuh-remoted",pid=XXXX,fd=Y))
tcp LISTEN 0 128 0.0.0.0:1514    0.0.0.0:* users:(("wazuh-remoted",pid=ZZZZ,fd=W))

🔍 Que pfSense esté enviando

En el manager:
    tcpdump -n udp port 514


Si ves líneas:
192.168.30.1.514 > 192.168.30.2.514: SYSLOG ...

✅ pfSense → Wazuh (red OK).

4️⃣ Ver que Wazuh guarda los logs (archives)

Revisa los archivos:
    ls -l /var/ossec/logs/archives/


Deberías ver:
- `archives.log`
- `archives.json`

Busca logs desde pfSense:
    grep 192.168.30.1 /var/ossec/logs/archives/archives.log | head


Si sale algo tipo:
2025 Nov 21 12:22:16 siem->192.168.30.1 Nov 21 12:22:16 filterlog[56916]: 4,,,1000000103,em0,match,block,in,...

✅ Wazuh ya está recibiendo y guardando los logs de pfSense.

5️⃣ Enviar archives al indexer con Filebeat

Para poder verlos en el dashboard, Filebeat debe enviar `archives.json` al Indexer.

Edita:
    sudo nano /etc/filebeat/filebeat.yml

Busca el módulo Wazuh, y asegúrate de que está así:
filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: true <-- Camiar a true

Guarda y reinicia Filebeat:
sudo systemctl restart filebeat
sudo systemctl status filebeat

7️⃣ Crear el Data View `wazuh-archives-*` (solo una vez)

En el Dashboard:

1. **Stack Management → Data Views (o Index patterns)**
2. **Create data view**
3. Nombre: `wazuh-archives`
4. Pattern / Index: `wazuh-archives-*`
5. Time field: selecciona `@timestamp`
6. Guardar.

Ahora, en **Discover** puede seleccionar `wazuh-archives-*`.

8️⃣ Ver los logs de pfSense en Discover (no alertas todavía)

En **Discover**:

1. Selecciona arriba el índice: `wazuh-archives-*`
2. En la barra de búsqueda, algo como:

    location: "192.168.30.1"

    
3. Añade a la tabla el campo `full_log`.

Verás líneas completas de pfSense tipo:

Nov 23 03:55:43 filterlog[57871]: 4,,,1000000103,em0,match,block,in,4,0x0,,1,13541,0,none,2,igmp,32,192.168.1.161,224.0.0.251,datalength=8


👉 En este punto **ya estás viendo los logs de pfSense** en Wazuh (a nivel de archivos).

***************************************************************************************************************
************************************************ DECODERS *****************************************************
***************************************************************************************************************

1. El Decoder (Copia y Pega en `/var/ossec/etc/decoders/pfsense_custom.xml`)

    nano /var/ossec/etc/decoders/pfsense_custom.xml

<decoder name="pfsense-custom-header">
    <prematch>^filterlog[\d+]: </prematch>
</decoder>

<decoder name="pfsense-custom-ipv4">
    <parent>pfsense-custom-header</parent>
    <regex type="pcre2" offset="after_parent">^.*?,(\w+),(\w+),(\w+),(\w+),4,.*?,(\w+),\d+,(\d+\.\d+\.\d+\.\d+),(\d+\.\d+\.\d+\.\d+)</regex>
    <order>interface, reason, action, direction, protocol, srcip, dstip</order>
</decoder>

<decoder name="pfsense-custom-ports">
    <parent>pfsense-custom-header</parent>
    <regex type="pcre2" offset="after_parent">^.*?,(\w+),(\w+),(\w+),(\w+),4,.*?,(tcp|udp),\d+,(\d+\.\d+\.\d+\.\d+),(\d+\.\d+\.\d+\.\d+),(\d+),(\d+)</regex>
    <order>interface, reason, action, direction, protocol, srcip, dstip, srcport, dstport</order>
</decoder>

2. Logs para probar (Copia y Pega en Logtest)

Aquí tienes 3 líneas distintas. Pégalas una por una en `/var/ossec/bin/wazuh-logtest` para verificar que **todo** funciona (ICMP, TCP y UDP).

Ejecuta: 
/var/ossec/bin/wazuh-logtest


**Línea 1: ICMP (Ping)**
Debe darte IPs y protocolo `icmp`.

filterlog[57871]: 97,,,1735123267,em0,match,pass,in,4,0x0,,128,25053,0,none,1,icmp,60,192.168.1.161,192.168.1.235,request,1,3740


**Línea 2: TCP (Navegación Web)**
Debe darte IPs, protocolo `tcp` y **puertos** (54321 y 80).

filterlog[57871]: 4,,,1000000103,em0,match,block,in,4,0x0,,1,1,0,DF,6,tcp,60,192.168.1.50,1.1.1.1,54321,80,0,S,12345,,1024,,

**Línea 3: UDP (Consulta DNS)**
Debe darte IPs, protocolo `udp` y **puertos** (12345 y 53).

filterlog[57871]: 5,,,1000000103,em0,match,pass,out,4,0x0,,64,12345,0,none,17,udp,76,192.168.1.161,8.8.8.8,12345,53,56


Reinicio Wazuh-Manager
systemctl restart wazuh-manager
systemctl status wazuh-manager

Habilitamos log PING en pfsense para pruebas.

3. Rules (Copia y Pega en `/var/ossec/etc/decoders/pfsense_custom.xml`)

nano /var/ossec/etc/rules/pfsense_rules.xml

<group name="pfsense,custom,">

  <rule id="100100" level="0">
    <decoded_as>pfsense-custom-header</decoded_as>
    <description>Evento base de pfSense</description>
  </rule>

  <rule id="100102" level="5">
    <if_sid>100100</if_sid>
    <action>pass</action>
    <protocol>icmp</protocol>
    <description>pfSense: Ping (ICMP) permitido desde $(srcip) a $(dstip)</description>
  </rule>

</group>

Reinicio Wazuh-Manager
systemctl restart wazuh-manager
systemctl status wazuh-manager
Mostrando WZ.12 - pfSense.txt.
