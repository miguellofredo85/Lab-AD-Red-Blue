- Baixar windows server
https://www.microsoft.com/es-es/evalcenter/download-windows-server-2022

- Baixar windows 10
https://www.microsoft.com/en-us/evalcenter

### Install server

- Mudar nome do DC para HYDRA-DC e restart

 Mudar server como DC

- Server Manager Dashboard → Manage → Add Role and Features
- Next
- Next
- Next
- On server role → checkbox AD domain services → Add Features → Next
- Next
- Next
- Install
- promote to domain controler (link)
- Add new forest name it MARVEL.local
- Next
- create pass → Netxt
- Next
- Next
- Next
- Install
- Close and sign in as Marvel Administrator

Mesmo para certificates (just check box for it then all next) continuar

- Click on configure AD Certificates Services link
- check box authority
- next
- next
- next
- next
- 99 years → next
- configure
- done e reboot

### No DC Server fazemos o seguinte para ter coneccao a internet 
- SYSTEM -> Network & Internet -> status -> Change Adapter options -> Ethernet properties -> TCP/IPv4 properties -> Check no Use the following DNS Server adresses -> Preferred DNS settings colocar ip 8.8.8.8


### Two join domain machines

- Agora criamos duas/treis maquinas windows 10

- skip private key section (por que nao temos)

- uncheck everything and install
  <img width="1085" height="736" alt="win inst-1" src="https://github.com/user-attachments/assets/b41fc694-f094-4649-868c-ccc0f2343ceb" />
<img width="1051" height="775" alt="win-instal-add-to-domain" src="https://github.com/user-attachments/assets/de21bfc5-41e4-46f0-b203-d8642c54031e" />
<img width="1025" height="698" alt="win-install-uncheck" src="https://github.com/user-attachments/assets/de5ca74f-167f-47be-b2f3-e67c0c92e722" />


- Escolher como exemplo THEPUNISHER usuario frankcastle e SPIDERMAN usuario peterparker. Depois do login mudar o nome da maquina (setting → about → rename this pc)
- SYSTEM -> Network & Internet -> status -> Change Adapter options -> Ethernet properties -> TCP/IPv4 properties -> Check no Use the following DNS Server adresses -> Preferred DNS settings colocar ip do DC 
<img width="868" height="622" alt="Netw-DC-2" src="https://github.com/user-attachments/assets/9bd3ca3d-4cdb-4c2f-9699-ca3343f55bde" />

- System -> About -> Rename this PC (advancel) -> Change -> Check no Domain e colocar o nome do dominio no meu caso foi MARVEL.local. Reiniciar o computador
- Pode conferir o computador vai aparecer no Server Dashboard -> Tools -> Users and Computer -> Computer

  <img width="1062" height="692" alt="Add-Computer-to-DC" src="https://github.com/user-attachments/assets/ed8da8fe-0e90-4515-93fc-caa94b1638fc" />

### Adicionar usuarios e/ou computadores no AD
- No server manager dashboard -> Tools -> AD Users and Computers

  - Exemplo Users
 click direito na pasta users -> new -> user -> comeca a inserir dados (em senha inserir a mesma do usuario original por exemplo a mesma do usuario fcastle e so check no PAssword never expires)
