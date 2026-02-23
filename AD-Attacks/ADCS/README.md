## Descrição
> Depois que a [SpectreOps](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf) divulgou o artigo científico Certified Pre-Owned, os Serviços de Certificados do Active Directory (AD CS) se tornaram um dos vetores de ataque preferidos dos agentes de ameaças por vários motivos, incluindo:
O uso de certificados para autenticação tem mais vantagens do que as credenciais comuns de nome de usuário/senha.
A maioria dos servidores PKI estava mal configurada/vulnerável a pelo menos um dos oito ataques descobertos pela SpectreOps (vários pesquisadores descobriram mais ataques desde então).
Há uma infinidade de vantagens em usar certificados e comprometer a Autoridade Certificadora (CA):
Os certificados de usuários e máquinas são válidos por mais de um ano.
Redefinir a senha de um usuário não invalida o certificado. Com os certificados, não importa quantas vezes um usuário altere sua senha; o certificado continuará válido (a menos que expire ou seja revogado).
Modelos mal configurados permitem a obtenção de um certificado para qualquer usuário.
Comprometer a chave privada da CA resulta na falsificação de certificados Golden.

