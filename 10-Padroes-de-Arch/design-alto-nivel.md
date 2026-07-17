# Design de Alto Nível

A ideia do Design de Alto Nível em uma entrevista de System Design é conseguir levantar uma ideia geral do sistema a ser desenvolvido, entendendo os requisitos e o contexto do projeto.

Exemplo: Se foi proposto para a gente que precisamos fazer um encurtador de URLs, o Design de Alto Nível seria entender quais são os requisitos e o contexto do projeto, e levantar uma ideia geral do sistema a ser desenvolvido.

Vamos supor que após perguntas para o entrevistador, conseguimos ter a ideia que:

- Precisamos encurtar URLs e lidar com algo pra não ter ids repetidos.
- Precisamos lidar com 10 milhões de URLs unicas
- Precisamos lidar com 1 milhão de requests

# Etapas

A primeira etapa seria pensar em formas de como lidar com a geração de URLs unicas, existem algumas formas pra fazer isso e a gente não precisa saber, apenas levantar uma ideia geral do que pode ser feito. Por exemplo existem:

- Hashing Function
- UUID
- Sequential IDs

Trabalhar com Sequential IDs pode não ser a melhor opção, pois pode gerar problemas de concorrência e de performance. Hash Functions e UUIDs são outras opções mais escaláveis. 

Depois vc precisa pensar o que vai ser salvo no banco e pensando nisso vc pode decidir qual opção é mais adequada para o projeto.

Pra esse caso, a gente precisaria salvar o Id, Url antiga e a Url encurtada (a url encurtada talvez nem precisaria se vc usa o id pra gerar essa url). Sabedo disso pra esse exemplo poderia usar tanto Banco relacional como noSQL, porem a ideia da escolha do banco e mostrar para o entrevistador qual opção seria mais adequada para o projeto dando um PORQUE.

Depois disso a ideia seria montar um System Design, comecando bem tranquilo, e escalando gradualmente conforme a identificação de gargalos, entao o primeiro design 

Exemplo: request -> server -> db. Um server nao aguenta as requests entao precisa adicionar mais servers, adicionando mais servers como vc lida com qual request vai pra onde, entao vc precisa de um load balancer para distrubuir a carga, e assim vc vai aumentando a capacidade do sistema.

![Alto Nível](../assets/alto-nivel.png)