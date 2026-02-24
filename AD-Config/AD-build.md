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


### Unir maquinas ao ominio

- Agora criamos duas/treis maquinas windows 10

- skip private key section (por que nao temos)

- uncheck everything and install
  <img width="1085" height="736" alt="win inst-1" src="https://github.com/user-attachments/assets/b41fc694-f094-4649-868c-ccc0f2343ceb" />
<img width="1051" height="775" alt="win-instal-add-to-domain" src="https://github.com/user-attachments/assets/de21bfc5-41e4-46f0-b203-d8642c54031e" />
<img width="1025" height="698" alt="win-install-uncheck" src="https://github.com/user-attachments/assets/de5ca74f-167f-47be-b2f3-e67c0c92e722" />


- Escolher como exemplo THEPUNISHER usuario frankcastle e SPIDERMAN usuario peterparker e alguma tipo SQLServer para logo brincar com o servicio. Depois do login mudar o nome da maquina (setting → about → rename this pc)
- SYSTEM -> Network & Internet -> status -> Change Adapter options -> Ethernet properties -> TCP/IPv4 properties -> Check no Use the following DNS Server adresses -> Preferred DNS settings colocar ip do DC 
<img width="868" height="622" alt="Netw-DC-2" src="https://github.com/user-attachments/assets/9bd3ca3d-4cdb-4c2f-9699-ca3343f55bde" />
<img width="1421" height="715" alt="ipdc" src="https://github.com/user-attachments/assets/39f893df-0ba7-4238-ad63-9462f8ed6c6e" />



- System -> About -> Rename this PC (advancel) -> Change -> Check no Domain e colocar o nome do dominio no meu caso foi MARVEL.local. Reiniciar o computador
- Pode conferir o computador vai aparecer no Server Dashboard -> Tools -> Users and Computer -> Computer
<img width="1062" height="692" alt="Add-Computer-to-DC" src="https://github.com/user-attachments/assets/ed8da8fe-0e90-4515-93fc-caa94b1638fc" />

### Adicionar usuarios e/ou computadores no AD
- No server manager dashboard -> Tools -> AD Users and Computers

  - Exemplo Users
 click direito na pasta users -> new -> user -> comeca a inserir dados (em senha inserir a mesma do usuario original por exemplo a mesma do usuario fcastle e check no PAssword never expires)

### Server Manager -> Tools -> Users and Computers
<img width="1249" height="503" alt="users" src="https://github.com/user-attachments/assets/a2f160e5-5011-4e24-aeab-2f791b4321c4" />
<img width="1876" height="562" alt="computers" src="https://github.com/user-attachments/assets/0655b1a5-bf06-46ff-9c7d-6f54b12cd002" />
<img width="1329" height="683" alt="grupos" src="https://github.com/user-attachments/assets/faff3c33-33c3-44b7-9b77-3c6a5539519f" />


## Habilitar a aba "Security" no ADUC
Por padrão, a aba "Security" (Segurança) não aparece nas propriedades de usuários no ADUC. Você precisa ativá-la primeiro:

- No HYDRA-DC, abra o Active Directory Users and Computers (dsa.msc).
- No menu superior, clique em View (Exibir) → Advanced Features (Recursos Avançados).
- Agora, ao abrir as propriedades de qualquer objeto, a aba Security estará visível.


















