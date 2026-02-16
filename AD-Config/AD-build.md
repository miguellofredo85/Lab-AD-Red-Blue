Baixar windows server
https://www.microsoft.com/es-es/evalcenter/download-windows-server-2022

Baixar windows 10
https://www.microsoft.com/en-us/evalcenter

install server

Mudar nome do DC para HYDRA-DC e restart

 Mudar server como DC

- Server Manager Dashboard → Maange → Add Role and Features
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

On DC Server 
- SYSTEM -> Network & Internet -> status -> Change Adapter options -> Ethernet properties -> TCP/IPv4 properties -> Check no Use the following DNS Server adresses -> Preferred DNS settings colocar ip 8.8.8.8


Two join domain machines

- Agora criamos duas maquinas windows 10

- skip private key section (por que nao temos)

- uncheck everything and install

- Escolher como exemplo THEPUNISHER usuario frankcastle e SPIDERMAN usuario peterparker. Depois do login mudar o nome da maquina (setting → about → rename this pc)
- SYSTEM -> Network & Internet -> status -> Change Adapter options -> Ethernet properties -> TCP/IPv4 properties -> Check no Use the following DNS Server adresses -> Preferred DNS settings colocar ip do DC 

- System -> About -> Rename this PC (advancel) -> Change -> Check no Domain e colocar o nome do dominio no meu caso foi MARVEL.local. Reiniciar o computador
- Pode conferir o computador vai aparecer no Server Dashboard -> Tools -> Users and Computer -> Computer

