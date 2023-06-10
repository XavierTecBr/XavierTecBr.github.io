---
title: Como estruturar o banco de dados para um processo de doações em servidores de game?
date: 2023-06-10 17:00:00 +/-TTTT
categories: [Banco de Dados, Como Fazer, Game]
tags: [banco de dados, game]     # TAG names should always be lowercase
author: <xavier_id>  
---

Bem pessoal, nessa oportunidade eu quero mostrar a vocês uma situação que eu me deparei algum tempo atrás.
Para quem não me conhece, além do meu trabalho de tempo integral automatizando processos de banco de dados num grande banco brasileiro, estudar Pós Graduação em Inteligência Artificial, escrever esse blog para compartilhar com vocês, também sou Dev voluntário num servidor mod de Role Play do famoso jogo Red Dead Rendemption 2, ou seja, Read Dead RP nas horas vagas, obviamente.

## Entendendo a Necessidade

Pois bem, dito isto a dificuldade era a seguinte: 
> Precisamos armazenar as doações que as pessoas fazem, mas só isso. Também precisamos que ao ocorrer a renovação dessa doação o player ganhe mais bônus de acordo com a quantidade de renovações que ela efetuar ao longo do tempo.
>
>Um detalhe importante é que existem alguns tipos de doações, onde cada uma delas possui nome e valor específico respectivamente. 
>
>Então, ao ocorrer uma nova doação é necessário que o sistema identifique baseado no nome da doação quantas vezes esse player já renovou esse mesmo plano de doação que o mesmo acabara de comprar.
>
>Pode ocorrer do player realizar uma compra antecipada, ou seja, antes do prazo de expiração do seu bônus acabar, porém o sistema deve reconhecer apenas a doação que está de acordo com a data hora corrente do servidor, evitando assim do player realizar várias doações e ganhar vários bônus simultaneamente, ao invés disso, ele só vai ganhar o bônus da segunda doação, após a primeira doação expirar.

Agora já temos o problema, agora depois de entender a necessidade vamos fazer alguns casos de uso e entender como essa regra vai se aplicar de uma forma geral no sistema com um todo, não só pensando apenas no banco de dados.

>Vamos assumir hipoteticamente que os pacotes são: Bronze-R$10, Prata-R$20 e Ouro-R$30

- Cenário 1:
    > Player José realizou uma doação escolhendo o pacote Bronze.
    >
    > Resultado esperado no jogo: Player José, Doador Bronze - Renovações 0x

- Cenário 2:
    > Player José após finalizar o seu bônus, realizou uma nova doação escolhendo o pacote Prata.
    >
    > Resultado esperado no jogo: Player José, Doador Prata - Renovações 0x

- Cenário 3:
    > Player José após finalizar o seu bônus, realizou uma nova doação escolhendo o pacote Bronze novamente.
    >
    > Resultado esperado no jogo: Player José, Doador Bronze - Renovações 1x

- Cenário 4:
    > Player José após finalizar o seu bônus, realizou uma nova doação escolhendo o pacote Prata novamente.
    >
    > Resultado esperado no jogo: Player José, Doador Prata - Renovações 1x

- Cenário 5:
    > Player José após finalizar o seu bônus, realizou uma nova doação escolhendo o pacote Bronze novamente.
    >
    > Resultado esperado no jogo: Player José, Doador Bronze - Renovações 2x

- Cenário 6:
    > Player José antes finalizar o seu bônus, realizou uma nova doação, desta vez escolhendo 2 pacotes Bronze, de uma vez.
    >
    > Resultado esperado no jogo: Player José, Doador Bronze - Renovações 2x
    > Após a expiração do bônus do Cenário 5, o sistema deverá reconhecer qual é a próxima doação a ser computada.
    > Assim o resultado será: Player José, Doador Bronze - Renovações 3x - Com mais uma doação a ser computada.

## Modelando o banco de dados

Entendido o problema e os cenários, fica mais fácil decidir como serão armazenados esses dado no banco de dados. Para este projeto utilizamos MariaDB então vamos ao menos criar um esboço de diagrama ER (Entidade Relacionamento) para chegarmos a uma solução final do armazenamento desses dados.

>Obviamente haveriam varias formas de resolver essa questão, o objetivo aqui não é dizer que a minha é melhor, porém é a que utilizei e resolveu o problema que o mais importante.

![Tabela Doações](renovacao-vip-games/renovacao-vip-games-foto-1.png)