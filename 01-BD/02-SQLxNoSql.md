# SQL x NoSQL

A principal diferença entre bancos SQL e NoSQL é que bancos SQL, em geral, priorizam ACID, enquanto bancos NoSQL, em geral, priorizam BASE. A ideia é explicar as siglas, quais cenários BDs relacionais são melhores, quais cenários BDs não relacionais são melhores e quais são os trade-offs.

## SQL

Basicamente, ACID é: Atomicidade, Consistência, Isolamento e Durabilidade. Vamos exemplificar cada uma dessas propriedades:

- **A — Atomicidade (Atomicity):** tudo ou nada. Se uma transação envolve atualizar o saldo e registrar um extrato, ou ambas as operações acontecem com sucesso, ou o banco desfaz tudo (*rollback*).

- **C — Consistência (Consistency):** o banco garante que o dado sairá de um estado válido para outro estado válido, respeitando todas as regras, chaves estrangeiras e *constraints* (ex.: saldo não pode ser negativo).

- **I — Isolamento (Isolation):** transações concorrentes não interferem umas nas outras. O resultado de rodar duas transações ao mesmo tempo deve ser equivalente ao de rodar uma depois da outra.

- **D — Durabilidade (Durability):** uma vez que a transação foi confirmada (*commit*), o dado está salvo de forma permanente, mesmo que falte energia no servidor um milissegundo depois.

---

## NoSQL

Bancos NoSQL (MongoDB, Cassandra, DynamoDB) surgiram para ajudar a resolver problemas de escala horizontal e distribuição global. Manter ACID estrito em uma rede com centenas de servidores espalhados pelo mundo pode ser lento demais. O BASE abre mão da consistência imediata para ganhar velocidade e tolerância a falhas.


### Basically Available

- Cada requisição recebe uma resposta
- Nem sempre os dados vão estar 100% atualizados
- Prefere disponibilidade e uptime à consistência
- Utilizado em sistemas distribuídos

Para lembrar disso, podemos pensar no Instagram, onde a gente tem um post que pode ser acessado em nível global. Se eu tenho informação vindo da China, Índia, Brasil e Austrália, fica difícil o número de likes dessa publicação ser sempre o número real de likes. Ele vai ter um número bem próximo do real, mas nem sempre vai ser 100% exato.

---

### Soft State

- O sistema pode mudar mesmo sem input
- Os dados podem estar temporariamente inconsistentes

Normalmente isso ocorre em cenários de sistemas distribuídos. Quando temos, por exemplo, um PostgreSQL no Brasil e outro na Austrália, e precisamos sincronizar esses dados, essa sincronização não é instantânea; leva um tempo.

---

### Eventual Consistency

- Os dados convergem para consistência. Então, se a gente tem um vídeo que recebe muitos likes por dia no YouTube, esses dados não vão estar 100% atualizados o tempo todo, porém aquele valor tende a ficar muito próximo do real. Quando esse vídeo parar de receber likes, os dados tendem a convergir para o valor correto.
- Consistência propagada de forma assíncrona


## Trade-off

BASE relaxa as restrições de ACID.

No teorema CAP, em sistemas distribuídos, não dá para garantir os 3 ao mesmo tempo quando há partição de rede:

1. Consistência.
2. Tolerância a partições.
3. Disponibilidade.

Por exemplo: em um banco distribuído, se acontecer uma partição de rede, o sistema normalmente precisa escolher entre continuar respondendo mesmo com risco de dado desatualizado (disponibilidade) ou bloquear/recusar parte das operações para preservar consistência.

A gente vai falar mais sobre CAP posteriormente.
