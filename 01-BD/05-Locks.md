# Locks

Locks são usados em casos nos quais temos processos concorrentes, assim como em sistemas operacionais, e precisamos controlar o acesso a um recurso compartilhado, como uma linha ou tabela.

Por exemplo, vamos pensar em um marketplace no qual o estoque de um produto é igual a 1. Se dois usuários tentarem comprá-lo ao mesmo tempo, uma condição de corrida pode permitir as duas compras e deixar o estoque negativo. Locks são uma das formas de evitar esse problema.

Existem duas abordagens comuns: lock pessimista e lock otimista.

## Lock pessimista

O lock pessimista bloqueia o recurso compartilhado até que a transação que o está usando termine. Se outra pessoa tentar fazer a compra do exemplo, sua transação precisará esperar. Na prática, no banco de dados, a trava pode ser feita assim:

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

### Vantagens do lock pessimista

- Evita atualizações perdidas.
- Facilita a preservação da consistência em cenários de alta contenção.
- Torna explícito quem pode alterar o recurso naquele momento.

### Desvantagens do lock pessimista

- Deadlocks. Por exemplo, uma transação trava o preço e espera o estoque, enquanto outra trava o estoque e espera o preço. O banco precisa detectar o ciclo e abortar uma delas.
- Recursos podem ficar bloqueados durante a espera.
- Menor escalabilidade quando há muita contenção.
- Transações longas aumentam o tempo de bloqueio.


## Lock otimista

O lock otimista permite que outros processos acessem o recurso. Em vez de manter uma trava durante toda a operação, ele usa uma versão para detectar se o registro mudou desde a leitura. A atualização inclui o valor esperado da versão; a primeira transação consegue alterá-lo e a segunda percebe que sua versão ficou desatualizada.

```sql
UPDATE produtos
SET estoque = estoque - 1,
    versao = versao + 1
WHERE id = 1
  AND estoque > 0
  AND versao = 7;
```

Se nenhuma linha for atualizada, a aplicação sabe que houve conflito e pode reler o registro ou tentar novamente.


### Vantagens do lock otimista

- Não mantém o recurso bloqueado durante todo o fluxo.
- Escala bem quando os conflitos são raros.
- Mantém uma vazão alta de leituras.

### Desvantagens do lock otimista

- Exige lógica de retry ou tratamento do conflito.
- Pode desperdiçar trabalho quando muitos processos disputam o mesmo dado.
- Adiciona complexidade à aplicação.
