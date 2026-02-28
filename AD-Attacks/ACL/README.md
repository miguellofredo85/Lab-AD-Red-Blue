Vou descrever apenas estas duas ACLs. Por quê? Não faz sentido descrever as demais, pois a maioria delas já dá uma ideia do que pode ser feito apenas pelo nome. Além disso, em sites como hacktricks ou no mesmo bloohound, por exemplo, é fácil encontrar como explorar cada uma delas. Então, por que apenas estas? Porque são difíceis de configurar no AD e optei por descrever cada uma delas detalhadamente. Se você quiser praticar com outras ACLs, basta ir para a seguinte configuração:

No DC, abrimos o Server Manager -> Users and Computers -> Users (ou Computers dependendo sua escolha) -> click direito sobre sua escolha -> Properties -> Security -> Advanced -> Select Principal (que vai ter a ACL sobre o usuario) -> escolha a ACL

. Aqui como exemplo colocaremos a ACL All Extended Rights


> <img width="1744" height="871" alt="hgfdjghjhgjfhgk" src="https://github.com/user-attachments/assets/53bf774c-b136-4249-ad63-630e9b01661e" />


> <img width="1822" height="801" alt="ert" src="https://github.com/user-attachments/assets/9d854110-4f92-42c5-be16-ab4d925e3382" />
