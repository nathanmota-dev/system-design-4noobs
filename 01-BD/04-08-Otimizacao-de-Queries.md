# 8. Otimização de Queries

Além da estrutura do banco, também é importante otimizar as queries.

Uma query mal escrita pode continuar lenta mesmo com bons índices.

Algumas estratégias importantes:

---

## 8.1. Usar EXPLAIN / EXPLAIN ANALYZE

`EXPLAIN` mostra como o banco pretende executar a query.

`EXPLAIN ANALYZE` executa a query e mostra o plano real, incluindo tempo gasto em cada etapa.

Isso ajuda a identificar problemas como:

* full table scan;
* índice não utilizado;
* join caro;
* ordenação pesada;
* leitura de muitas linhas desnecessárias.

Exemplo:

```sql
EXPLAIN ANALYZE
SELECT * FROM orders
WHERE user_id = 10
ORDER BY created_at DESC;
```

---

## 8.2. Filtrar cedo

Sempre que possível, é melhor reduzir o volume de dados antes de fazer joins ou agregações.

Exemplo ruim:

```sql
SELECT *
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE o.created_at >= '2026-01-01';
```

Dependendo do banco e do plano de execução, pode ser melhor garantir que o filtro reduza os dados o quanto antes.

Exemplo conceitual:

```sql
SELECT *
FROM (
  SELECT * FROM orders
  WHERE created_at >= '2026-01-01'
) o
JOIN users u ON u.id = o.user_id;
```

Na prática, otimizadores modernos muitas vezes já fazem isso, mas em entrevistas é válido mencionar a ideia: reduzir o volume de dados o mais cedo possível.

---

## 8.3. Evitar nested subqueries desnecessárias

Subqueries aninhadas podem deixar a query mais difícil de otimizar e entender.

Nem toda subquery é ruim, mas subqueries desnecessárias podem piorar performance.

Às vezes, uma query pode ser reescrita usando:

* joins;
* CTEs;
* índices melhores;
* materialized views;
* tabelas auxiliares.

---

## 8.4. Paginação

Retornar muitos dados de uma vez é um problema comum.

Exemplo ruim:

```sql
SELECT * FROM orders;
```

Em uma tabela grande, isso pode gerar:

* alto consumo de memória;
* resposta lenta;
* sobrecarga na rede;
* pior experiência para o usuário.

O ideal é paginar.

Exemplo com `LIMIT` e `OFFSET`:

```sql
SELECT * FROM orders
ORDER BY created_at DESC
LIMIT 20 OFFSET 100;
```

Porém, em tabelas muito grandes, paginação com `OFFSET` pode ficar lenta.

Uma alternativa é usar **cursor-based pagination**.

Exemplo:

```sql
SELECT * FROM orders
WHERE created_at < '2026-06-01T10:00:00'
ORDER BY created_at DESC
LIMIT 20;
```

Cursor pagination costuma escalar melhor para feeds, timelines e listas grandes.

---

## 8.5. Batching

Batching consiste em agrupar várias operações em uma única chamada ou transação.

Exemplo ruim:

```txt
Inserir 1 registro por request.
Inserir 1 registro por request.
Inserir 1 registro por request.
```

Exemplo melhor:

```txt
Inserir 100 registros em uma única operação.
```

Isso reduz overhead de rede, transações e conexões.

Batching é útil para:

* inserção em massa;
* processamento assíncrono;
* importação de dados;
* atualização de muitos registros;
* sistemas com eventos.

---

## 8.6. Materialized Views

Uma materialized view armazena o resultado de uma query previamente calculada.

Isso é útil para queries caras que não precisam ser recalculadas a todo momento.

Exemplo:

```txt
Relatório mensal de vendas.
Dashboard com métricas agregadas.
Ranking diário.
```

Se os dados não mudam com frequência, faz sentido pré-computar o resultado.

Exemplo:

```sql
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT
  date_trunc('month', created_at) AS month,
  SUM(total) AS total_sales
FROM orders
GROUP BY month;
```

Vantagens:

* leitura muito mais rápida;
* bom para dashboards e relatórios;
* reduz queries pesadas em tempo real.

Desvantagens:

* dados podem ficar desatualizados;
* precisa definir estratégia de refresh;
* ocupa armazenamento extra.

---
