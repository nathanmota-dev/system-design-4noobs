# 5. Sharding e Partitioning

Em bancos com grande volume de dados, uma única tabela ou um único servidor pode não ser suficiente.

Nesses casos, podemos dividir os dados para melhorar:

* latência;
* tempo de consulta;
* throughput de leitura;
* throughput de escrita;
* distribuição de carga;
* capacidade de armazenamento.

Duas estratégias comuns são:

* **partitioning**;
* **sharding**.

Apesar de serem parecidas, não são exatamente a mesma coisa.

---

## 5.1. Partitioning

Partitioning é a divisão lógica dos dados dentro do próprio banco.

Em vez de manter uma tabela gigante como uma única estrutura, o banco pode dividi-la em partes menores.

Exemplo: particionar uma tabela de logs por mês.

```txt
logs_2026_01
logs_2026_02
logs_2026_03
```

Ou, em uma tabela particionada:

```txt
logs
 ├── partição janeiro
 ├── partição fevereiro
 └── partição março
```

Isso ajuda porque uma query pode acessar apenas a partição necessária.

Exemplo:

```sql
SELECT * FROM logs
WHERE created_at BETWEEN '2026-01-01' AND '2026-01-31';
```

Se a tabela for particionada por data, o banco pode consultar apenas a partição de janeiro.

Tipos comuns de partitioning:

* **por faixa/range**: exemplo, datas ou valores numéricos;
* **por hash**: distribui os dados usando uma função hash;
* **por lista/list**: separa por valores específicos, como país ou categoria.

---

## 5.2. Sharding

Sharding é a divisão dos dados entre múltiplos bancos ou servidores.

Enquanto partitioning costuma acontecer dentro do mesmo banco, sharding normalmente distribui os dados em diferentes nós.

Exemplo:

```txt
Shard 1 -> usuários de A até F
Shard 2 -> usuários de G até M
Shard 3 -> usuários de N até Z
```

Ou usando hash:

```txt
hash(user_id) % número_de_shards
```

O objetivo é distribuir carga e armazenamento horizontalmente.

## Estratégias comuns de sharding

### Sharding por faixa

Divide os dados por intervalos.

Exemplo:

```txt
user_id 1 até 1.000.000       -> Shard 1
user_id 1.000.001 até 2.000.000 -> Shard 2
```

Vantagem:

* simples de entender;
* bom para queries por intervalo.

Desvantagem:

* pode gerar hot spots se muitos acessos ficarem concentrados em uma faixa.

---

### Sharding por hash

Usa uma função hash para distribuir os dados.

Exemplo:

```txt
hash(user_id) % 4
```

Vantagem:

* distribui melhor os dados;
* reduz chance de hot spots.

Desvantagem:

* queries por intervalo ficam mais difíceis;
* adicionar novos shards pode exigir rebalanceamento.

---

### Sharding por chave ou tenant

Divide os dados com base em uma chave de negócio.

Exemplo:

```txt
empresa A -> Shard 1
empresa B -> Shard 2
empresa C -> Shard 3
```

Isso é comum em sistemas multi-tenant.

Vantagem:

* isolamento por cliente/tenant;
* pode facilitar escala por organização.

Desvantagem:

* tenants muito grandes podem concentrar carga em um único shard.

---

## Trade-offs de sharding

Sharding melhora escala, mas aumenta bastante a complexidade.

Problemas comuns:

* queries entre shards são mais difíceis;
* transações distribuídas ficam complexas;
* joins entre shards são ruins;
* rebalanceamento pode ser difícil;
* a aplicação precisa saber onde buscar os dados;
* consistência pode ser mais difícil de garantir.

Em entrevista de System Design, sharding costuma ser uma solução para escala alta, mas não deve ser a primeira opção. Normalmente, antes de sharding, pensamos em:

1. bons índices;
2. otimização de queries;
3. cache;
4. read replicas;
5. partitioning;
6. sharding.

---
