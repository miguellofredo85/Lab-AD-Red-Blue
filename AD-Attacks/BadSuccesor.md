👉 [Explicação](#explicação)  
⚙️ [Configuração](#configuração)  
⚠️ [Ataque](#ataque)  
🛡️ [Prevenção](#prevenção)  
📊 [Detecção](#detecção)

---

## Explicação
Imagina que você trabalha em uma empresa bem grande, com muitos computadores e servidores. Para organizar essa bagunça, o pessoal de TI usa um "livro de regras" central chamado Active Directory, que basicamente diz quem pode acessar o quê na rede.

A Microsoft lançou uma novidade para versão mais nova do servidor (Windows Server 2025) chamada dMSA. Pense nisso como um "cartão de acesso temporário" que você pode dar para um programa de computador, para que ele possa executar suas tarefas sem precisar da senha de um usuário de verdade. É uma ideia útil para automatizar coisas.

Agora, o "BadSuccessor" é um truque sujo que os invasores descobriram para usar essa novidade a favor deles.

A ideia é mais ou menos assim:

O Ponto de Partida: O invasor já conseguiu entrar na rede da empresa, mas com um acesso bem limitado. Ele é como um estagiário que só tem permissão para mexer em alguns arquivos bobos.

A Brecha (Permissão de "Jardineiro"): Acontece que, na configuração da empresa, esse estagiário (o invasor) tem permissão para criar pastas ou "caixas organizadoras" em um setor específico da empresa. É como se ele pudesse decidir onde colocar as plantas no jardim, mas não pudesse entrar na sala do diretor.

Criando o Cavalo de Troia: O invasor então cria um desses "cartões de acesso temporário" (o dMSA) e o coloca em uma dessas pastas que ele pode administrar. Mas aqui está o pulo do gato: ele configura esse cartão para imitar o cartão de acesso do diretor da empresa (a conta de "Domínio Admin", que tem acesso a tudo).

A Mágica (Herança de Poder): Por causa de uma peculiaridade no funcionamento desse sistema de permissões (o Kerberos, que é o porteiro que distribui os passes), esse novo cartão falso acaba herdando todos os poderes do diretor.

O Resultado: Agora, com esse cartão falso em mãos, o invasor pode pedir ao porteiro (Kerberos) passes de entrada para qualquer sala da empresa, como se fosse o próprio diretor. Ele consegue acesso total a tudo.

Por que esse truque é tão perigoso?

Silencioso: Ele não usa aquelas técnicas barulhentas e conhecidas, como tentar adivinhar senhas ou forjar um "passe mestre" (golden ticket). Ele simplesmente usa as ferramentas que já existem no sistema do jeito errado.

Aproveitador: O invasor não precisou invadir a conta do diretor. Ele só criou um novo "cartão" e disse que ele era igual ao do diretor. O sistema aceitou.

Difícil de Perceber: Como ele está usando funcionalidades legítimas do Windows, os sistemas de segurança podem não achar isso estranho.

Em resumo, o BadSuccessor é um golpe de identidade. Um invasor com poucos poderes consegue criar uma identidade falsa que se passa por um superusuário, usando para isso uma falha na configuração de uma funcionalidade nova do sistema.

...

## Configuração

...

## Ataque

...

## Prevenção

...

## Detecção
