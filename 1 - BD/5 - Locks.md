### Locks

Locks são usados para casos onde temos processos concorrentes (assim como em SOs) e precisamos garantir que apenas um processo possa acessar um recurso compartilhado (como uma tabela) por vez. 

Exemplo, vamos pensar no mercado livre onde os vendedores tem a quantidade de produtos em estoque, se estoque = 1 e dois usuarios tentam fazer essa compra ao mesmo momento, sem pensar em lock eles consegueriam comprar o produto e o estoque ficaria negativo, lock veio para resolver esse problema.

Dentro de Locks temos o Lock Pessimista e o Lock Otimista. 

O Lock Pessimista bloqueia o recurso compartilhado até que o processo que o está usando termine, então se tiver outra pessoa tentando fazer a compra conforme aquele exemplo que a gente fez, ela não consegue acessar o recurso até que o primeiro termine. Na prática, no Banco de dados acontece uma trava como

```sql
SELECT * FROM produtos WHERE id = 1 FOR UPDATE;
```

esse for update trava o recurso até que o primeiro processo termine, quando o usuario compra o produto, o estoque é atualizado e o lock é liberado atraves do commit, permitindo que outros processos acessem o recurso.

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
