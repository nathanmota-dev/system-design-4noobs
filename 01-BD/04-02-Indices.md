## 2. Índices

Índices são uma das otimizações mais importantes em bancos de dados. mUm índice funciona como uma estrutura auxiliar que permite encontrar dados mais rapidamente sem precisar varrer a tabela inteira. Sem índice, uma busca pode exigir um **full table scan**:

```sql
SELECT * FROM users WHERE email = 'user@email.com';
```

Se a tabela tiver milhões de usuários e não existir índice em `email`, o banco pode precisar verificar muitas linhas até encontrar o resultado.Com índice, o banco consegue localizar o dado de forma muito mais eficiente.

### Quando usar índices?

Índices fazem sentido quando uma coluna é usada com frequência em:

* filtros com `WHERE`;
* ordenações com `ORDER BY`;
* joins;
* buscas por igualdade;
* buscas por intervalo;
* colunas usadas em constraints, como `UNIQUE`.

Exemplo:

```sql
CREATE INDEX idx_users_email ON users(email);
```

Isso melhora consultas como:

```sql
SELECT * FROM users WHERE email = 'user@email.com';
```

### Trade-offs dos índices

Índices melhoram leitura, mas têm custo.

Os principais pontos negativos são:

* escritas mais lentas, porque o índice também precisa ser atualizado;
* maior uso de armazenamento;
* maior complexidade na escolha dos índices;
* índices desnecessários podem atrapalhar a performance geral.

Por isso, não faz sentido criar índice para toda coluna. O ideal é criar índices com base nos padrões reais de acesso.

---

### 2.1. Índice B-tree

O índice **B-tree** é o tipo mais comum em bancos relacionais.

Ele funciona bem para:

* buscas por igualdade;
* ordenação;
* ranges;
* comparações com `<`, `>`, `<=`, `>=`;
* filtros por datas ou números.

Exemplo:

```sql
SELECT * FROM orders
WHERE created_at BETWEEN '2022-01-01' AND '2026-01-01';
```

Para esse tipo de consulta por intervalo, um índice B-tree costuma fazer bastante sentido.

---

### 2.2. Índice Hash

Índices do tipo **hash** são úteis para buscas exatas por igualdade.

Exemplo:

```sql
SELECT * FROM users WHERE email = 'user@email.com';
```

Eles podem ser eficientes quando a consulta é sempre algo como:

```sql
coluna = valor
```

Porém, hash index não é adequado para buscas por intervalo ou ordenação.

Exemplo ruim para hash:

```sql
SELECT * FROM orders WHERE created_at > '2025-01-01';
```

Nesse caso, B-tree costuma ser mais adequado.

---

### 2.3. Índice composto

Um índice composto envolve mais de uma coluna.

Exemplo:

```sql
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
```

Esse índice pode ajudar em queries como:

```sql
SELECT * FROM orders
WHERE user_id = 10 AND status = 'paid';
```

A ordem das colunas importa. Um índice em `(user_id, status)` não é necessariamente equivalente a um índice em `(status, user_id)`.

Índices compostos são muito comuns em sistemas reais, principalmente quando as queries filtram por múltiplos campos.

---

### 2.4. Outros tipos de índice

Além de B-tree, hash e composto, existem outros tipos importantes dependendo do banco e do caso de uso:

* **Full-text index**: usado para busca textual.
* **GIN**: comum no PostgreSQL para arrays, JSONB e full-text search.
* **GiST**: usado para dados geométricos, buscas aproximadas e alguns tipos específicos.
* **BRIN**: útil para tabelas muito grandes com dados naturalmente ordenados, como logs por data.

---

