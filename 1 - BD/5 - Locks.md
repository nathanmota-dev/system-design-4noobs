### Locks

Locks são usados para casos onde temos processos concorrentes (assim como em SOs) e precisamos garantir que apenas um processo possa acessar um recurso compartilhado (como uma tabela) por vez. 

Exemplo, vamos pensar no mercado livre onde os vendedores tem a quantidade de produtos em estoque, se estoque = 1 e dois usuarios tentam fazer essa compra ao mesmo momento, sem pensar em lock eles consegueriam comprar o produto e o estoque ficaria negativo, lock veio para resolver esse problema.

Dentro de Locks temos o Lock Pessimista e o Lock Otimista. 

O Lock Pessimista bloqueia o recurso compartilhado até que o processo que o está usando termine, então se tiver outra pessoa tentando fazer a compra conforme aquele exemplo que a gente fez, ela não consegue acessar o recurso até que o primeiro termine. Na prática, no Banco de dados acontece uma trava como

```sql
BEGIN;

SELECT id, nome, estoque
FROM produtos
WHERE id = 1
FOR UPDATE;

UPDATE produtos
SET estoque = estoque - 1
WHERE id = 1
  AND estoque > 0;

COMMIT;
```

Nesse exemplo:

- A transação começa no `BEGIN`.
- A trava acontece na linha do `FOR UPDATE`. Quando o banco executa esse `SELECT`, ele coloca um lock de escrita na linha retornada (`id = 1`).
- Enquanto essa transação estiver aberta, outra transação que tentar dar `SELECT ... FOR UPDATE`, `UPDATE` ou `DELETE` nessa mesma linha vai ficar esperando.
- O `UPDATE` altera o estoque enquanto a linha continua travada pela transação atual.
- A liberação da trava acontece no `COMMIT` (ou no `ROLLBACK`, se der erro). Nesse momento o banco confirma a alteração e solta o lock para os próximos processos.

Na prática, com duas sessões concorrentes, ficaria assim:

**Sessão A**

```sql
BEGIN;

SELECT id, nome, estoque
FROM produtos
WHERE id = 1
FOR UPDATE;

-- a linha do produto id = 1 ficou travada aqui

UPDATE produtos
SET estoque = estoque - 1
WHERE id = 1
  AND estoque > 0;

-- a trava ainda continua ativa aqui

COMMIT;

-- a trava só é liberada aqui
```

**Sessão B** (executando ao mesmo tempo)

```sql
BEGIN;

SELECT id, nome, estoque
FROM produtos
WHERE id = 1
FOR UPDATE;

-- essa query fica esperando a Sessão A dar COMMIT ou ROLLBACK

UPDATE produtos
SET estoque = estoque - 1
WHERE id = 1
  AND estoque > 0;

COMMIT;
```

Ou seja, a Sessão B não consegue "passar na frente". Ela fica bloqueada exatamente na linha do `SELECT ... FOR UPDATE`, porque a Sessão A já travou esse registro. Depois que a Sessão A faz `COMMIT`, a Sessão B continua a execução, lê o valor mais recente do estoque e só então decide se ainda consegue atualizar.

Vantagens do Lock Pessimista:
- Segurança
- Simples
- Não tem updates perdidos
- Consistência

Desvantagens do Lock Pessimista:

- Deadlocks, isso ocorre quando por exemplo um processo precisa da tabela de preço e estoque e precisa travar os dois, então ele trava a tabela de preço porém o outro processo trava a tabela de estoque e fica esperando o primeiro terminar, mas nunca termina, então ocorre um deadlock.
- Bloqueio de recursos
- Escalabilidade
- Transações longas


Já o Lock Otimista permite que outros processos acessem o recurso enquanto o primeiro não termina, ou seja, ele não trava o recurso e pra saber que aquele processo foi atualizado ele utiliza um versionamento.
Como o update precisa atualizar dois dados agora que são o estoque e a versão, o primeiro que conseguir fazer a query consegue fazer, o segundo não consegue falando assim essa versão já foi atualizada. 


Vantagens do Lock Otimista:
- Não bloqueia o recurso
- Escalabilidade
- Vazão alta de reads

Desvantagens do Lock Otimista:
- Retries (precisa criar essa lógica pra quando der errado)
- Complexidade
