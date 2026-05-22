# Otimizações de DBs

A ideia dessa aula é abordar as diferentes otimizações de BDs a se pensar em uma entrevista de SD.

---

## 1 - Cache

Primeira forma de otimizar um banco de dados é cache, ou seja, jogar o gargalo de requisições em um Redis da vida, por exemplo. Porém, como o foco vai ser focar em cache mais pra frente, vamos pensar em estratégias sem isso por agora, mas normalmente a primeira coisa a se pensar é cache para diminuir o gargalo.

---

## 2 - Index (Índice)

A segunda forma de otimização de banco de dados mais comum é índice e temos tipos comuns de utilizar, como B-tree e hash. Índice é utilizado quando fazemos muitas buscas em um campo específico no banco de dados, como por exemplo: buscar um nome de usuário. Se isso é uma operação muito requisitada do banco de dados, faz sentido criar um índice para isso (nesse caso um hash pode fazer sentido, mas não é a única opção). Pontos negativos: writes mais lentos, por precisar atualizar o índice também e gastar mais armazenamento.

### B-tree
B-tree é o padrão mais comum quando estamos trabalhando com um range específico como 2022-2026

### Hash
O hash é mais rápido quando vamos buscar por uma igualdade, como email = x.

Existem outros tipos de index como: composto, full-text, GIN/GiST. Índice composto, por exemplo, também é bem comum.

---

## 3 - Replicação de Reads

Outra estratégia de otimização de banco de dados é fazer uma replicação apenas nos reads, essa estratégia funciona assim: vamos supor que estamos lidando com uma aplicação como Twitter ou LinkedIn, onde quando uma pessoa faz um post, esse post pode atingir várias pessoas. Vamos supor que a média é que 1 post atinge 100 pessoas. Nesse caso a gente tem um gargalo bem grande em read comparado a write, porque enquanto 1 pessoa publica um post, 100 pessoas leem o post, então a melhor estratégia nesse caso seria ter um banco de write e várias cópias desse BD apenas para read, onde está o maior gargalo da operação. Precisamos lembrar que ao lidar com múltiplos bancos de dados esses dados demoram para ficar disponíveis em todos os BDs de read, porém nesse caso não é algo tão importante porque não importa muito tweetar e o outro usuário demorar 15 segundos para ver o tweet. Porém, falando de aplicações como bancos, isso já seria totalmente diferente, onde o indicado seria assegurar consistência forte, por exemplo lendo do banco de write ou garantindo sincronização, mesmo que a request demore mais.

---

## 4 - Sharding e Partition



---

## 5 - Teorema de CAP



---

## 6 - Normalização e Denormalização

### Normalização

Normalização trata-se de dividir as informações de 1 tabela em duas ou mais tabelas para reduzir redundância e evitar inconsistência. Exemplo: vamos supor que uma tabela contém as informações de usuário e suas informações de conta bancária. A gente poderia dividir essa informação em duas tabelas para melhorar a performance caso precise apenas do conteúdo do usuário. O trade-off disso é que leituras que precisam das informações das duas tabelas ficam mais lentas (uso de join, por exemplo).

### Denormalização

Duplicar informação em tabelas diferentes, porém deixando a leitura mais rápida (não precisa de JOIN, por exemplo). Isso é ruim porque, se esquecer de atualizar os dados nas duas tabelas, os dados ficam inconsistentes.

---

## Outras estratégias

Agora vamos falar de outras estratégias menos usadas.

1 - Connection pooling

Essa estratégia é reaproveitar as conexões do banco de dados, um problema que acontece muito com Supabase, por exemplo, onde por padrão quando o usuário faz uma requisição ele é conectado ao banco de dados. O problema disso é que, conforme mais requisições acontecem, mais usuários abrem conexões diferentes, podendo ter um gargalo no volume de requisições. O que dá pra fazer é manter um pool de conexões para ser reutilizado entre várias requisições e usuários (no exemplo acima do Supabase, cada caso é um caso), dessa forma diminui o overhead em cada requisição.

2 - Otimização de queries

- Evitar nested subqueries
- Usar o EXPLAIN ANALYZE para ver onde pode ser melhorado
- Filtrar cedo para reduzir o volume de dados antes do JOIN, quando possível
- Materialized view (Ex: dados do mês passado, como já foi, não vai ser mais atualizado, faz sentido usar uma materialized view)
- Batching e pagination (paginação)

---
