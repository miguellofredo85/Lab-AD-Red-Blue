Baixar Sysmon 
https://technet.microsoft.com/en-us/sysinternals/sysmon


Criacao do arquivo sysconfig.xml 

<Sysmon schemaversion="3.30">
 <HashAlgorithms>md5</HashAlgorithms>
 <EventFiltering>
 <!--SYSMON EVENT ID 1 : PROCESS CREATION-->
 <ProcessCreate onmatch="include">
 <Image condition="contains">powershell.exe</Image>
 </ProcessCreate>
 <!--SYSMON EVENT ID 2 : FILE CREATION TIME RETROACTIVELY CHANGED IN THE FILESYSTEM-->
 <FileCreateTime onmatch="include"></FileCreateTime>
 <!--SYSMON EVENT ID 3 : NETWORK CONNECTION INITIATED-->
 <NetworkConnect onmatch="include"></NetworkConnect>
 <!--SYSMON EVENT ID 4 : RESERVED FOR SYSMON STATUS MESSAGES, THIS LINE IS INCLUDED FOR DOCUMENTATION PURPOSES ONLY-->
 <!--SYSMON EVENT ID 5 : PROCESS ENDED-->
 <ProcessTerminate onmatch="include"></ProcessTerminate>
 <!--SYSMON EVENT ID 6 : DRIVER LOADED INTO KERNEL-->
 <DriverLoad onmatch="include"></DriverLoad> 
 <!--SYSMON EVENT ID 7 : DLL (IMAGE) LOADED BY PROCESS-->
 <ImageLoad onmatch="include"></ImageLoad>
 <!--SYSMON EVENT ID 8 : REMOTE THREAD CREATED-->
 <CreateRemoteThread onmatch="include"></CreateRemoteThread>
 <!--SYSMON EVENT ID 9 : RAW DISK ACCESS-->
 <RawAccessRead onmatch="include"></RawAccessRead> 
 <!--SYSMON EVENT ID 10 : INTER-PROCESS ACCESS-->
 <ProcessAccess onmatch="include"></ProcessAccess>
 <!--SYSMON EVENT ID 11 : FILE CREATED-->
 <FileCreate onmatch="include"></FileCreate>
 <!--SYSMON EVENT ID 12 & 13 & 14 : REGISTRY MODIFICATION-->
 <RegistryEvent onmatch="include"></RegistryEvent>
 <!--SYSMON EVENT ID 15 : ALTERNATE DATA STREAM CREATED-->
 <FileCreateStreamHash onmatch="include"></FileCreateStreamHash> 
 <PipeEvent onmatch="include"></PipeEvent>
 </EventFiltering>
</Sysmon>


Instalar Sysmon
Sysmon64.exe -accepteula -i sysconfig.xml

**Configurar Wazuh agent para monitorar os eventos**
Na pasta de instalacao do Wazuh agent  (geralmente c:\Program Filex86\ossec-agent), abrir ossec.conf e pegar antes do fechamento do codigo

<localfile>
<location>Microsoft-Windows-Sysmon/Operational</location>
<log_format>eventchannel</log_format>
</localfile>

<img width="721" height="949" alt="wazuhsys" src="https://github.com/user-attachments/assets/adc3b604-6c75-474d-876e-248b9064c832" />
```
<localfile>
<location>Microsoft-Windows-Sysmon/Operational</location>
<log_format>eventchannel</log_format>
</localfile>
```


Reiniciar o servicio Wazuh

Restart-Service WazuhSvc


No server manager ir a /vat/ossec/etc/rules/local_rules.xml e colcar
<group name="sysmon,">
 <rule id="255000" level="12">
 <if_group>sysmon_event1</if_group>
 <field name="sysmon.image">\powershell.exe||\.ps1||\.ps2</field>
 <description>Sysmon - Event 1: Bad exe: $(sysmon.image)</description>
 <group>sysmon_event1,powershell_execution,</group>
 </rule>
</group>


Reiniciar service wazuh-manager e agent


Pode conferir na maquina windows a instalacao bem sucedida

Event Viewer → App and Services Logs→ Microsoft → Windows → Sysmon → Operactional
<img width="1751" height="793" alt="w2" src="https://github.com/user-attachments/assets/2ecd53fe-4aa4-448c-ab01-090470167cbe" />

No dashboard do Wazuh 

Active (chosse the agent) → Threath Hunting → Events
<img width="1830" height="636" alt="w3" src="https://github.com/user-attachments/assets/f42d2274-ee18-4529-8d9f-713aa35b38bb" />


