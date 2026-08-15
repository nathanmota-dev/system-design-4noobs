# 1. Cache

Uma das formas mais comuns de aliviar a carga do banco de dados é usar **cache**.

A ideia é evitar que toda requisição precise acessar diretamente o banco. Em vez disso, dados acessados com frequência podem ser armazenados temporariamente em uma camada mais rápida, como Redis ou Memcached.

Exemplo:

```txt
Usuário faz request -> 

API verifica Redis -> 

se existir no cache, retorna

-> se não existir, busca no banco e salva no cache
```

Isso é útil principalmente para dados muito lidos e pouco alterados, como:

* perfil público de usuário;
* catálogo de produtos;
* configurações;
* feed pré-computado;
* rankings;
* respostas de queries caras.

O principal benefício é reduzir a pressão no banco e diminuir a latência das respostas.

## Trade-offs de cache

Se o cache é usado para salvar uma consulta com vários joins e os dados de origem mudam, o valor armazenado pode ficar desatualizado. Para lidar com isso, existem três estratégias principais:

1. **Expiration:** os dados no cache são removidos após um tempo definido.
2. **Refresh:** os dados no cache são atualizados periodicamente.
3. **Delete:** os dados no cache são removidos quando a fonte original muda.

Outros trade-offs:

* invalidação de cache;
* dados desatualizados;
* maior complexidade na aplicação;
* necessidade de definir TTL;
* risco de cache stampede, quando várias requisições tentam recalcular o mesmo dado ao mesmo tempo.

---
