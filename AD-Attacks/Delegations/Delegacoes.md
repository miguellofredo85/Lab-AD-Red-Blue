A delegação Kerberos permite que um aplicativo acesse recursos hospedados em um servidor diferente; por exemplo, em vez de dar à conta de serviço que executa o servidor web acesso direto ao banco de dados, podemos permitir que a conta seja delegada ao serviço do servidor SQL. Quando um usuário faz login no site, a conta de serviço do servidor web solicita acesso ao serviço do servidor SQL em nome desse usuário, permitindo que ele acesse o conteúdo do banco de dados que lhe foi provisionado sem precisar atribuir nenhum acesso à conta de serviço do servidor web em si.
Como o nome sugere, a delegação irrestrita é a mais permissiva, permitindo que uma conta delegue a qualquer serviço. Na delegação restrita, uma conta de usuário terá suas propriedades configuradas para especificar quais serviços ela pode delegar. Para a delegação baseada em recursos, a configuração está dentro do objeto do computador para o qual a delegação ocorre. 


|Tipo de Delegação	| Onde Configura |	Quem Configura	| Risco | 	Ticket Forwardable	
|-------------|-------------|----------|------------|--------------|
|Irrestrita (Unconstrained)	| Conta de Serviço (Origem) | 	Admin do Domínio | 	🔴 Alto - Qualquer destino |	Sim	|
|Restrita (Constrained) | Conta de Serviço (Origem)	|Admin do Domínio|	🟡 Médio - Destinos específicos|	Com transição: Sim \\Sem transição: Não	|
|Baseada em Recursos (Resource-Based)|	Objeto do Recurso (Destino)	|Admin do Recurso (local)	|🟢 Baixo - Controle pelo destino|	Não necessário	|
