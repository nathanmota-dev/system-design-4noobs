# Otimizações de Banco de Dados

A ideia aqui é abordar as principais estratégias de otimização de banco de dados que podem aparecer em uma entrevista de System Design.

Quando falamos em otimizar um banco de dados, normalmente estamos tentando resolver problemas como:

* alta latência nas consultas;
* excesso de carga no banco;
* muitas conexões simultâneas;
* queries lentas;
* gargalos de leitura ou escrita;
* dificuldade de escalar o volume de dados.

É importante lembrar que toda otimização tem trade-offs. Uma estratégia pode melhorar leitura, mas piorar escrita. Pode reduzir latência, mas aumentar complexidade. Pode melhorar escala, mas dificultar consistência.

A partir dos próximos tópicos, vamos explorar cada uma dessas estratégias de otimização de banco de dados.

* [1. Cache](./04-01-Cache.md)
* [2. Índices](./04-02-Indices.md)
* [3. Connection Pooling](./04-03-Connection-Pooling.md)
* [4. Replicação de leitura](./04-04-Replicacao-de-Leitura.md)
* [5. Sharding e Partitioning](./04-05-Sharding-e-Partitioning.md)
* [6. Teorema CAP](./04-06-Teorema-CAP.md)
* [7. Normalização e Denormalização](./04-07-Normalizacao-e-Denormalizacao.md)
* [8. Otimização de Queries](./04-08-Otimizacao-de-Queries.md)
* [Resumo](./04-09-Resumo.md)
