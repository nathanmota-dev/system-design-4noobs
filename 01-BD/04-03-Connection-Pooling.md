# 3. Connection Pooling

Abrir uma conexão com o banco de dados tem custo.

Se cada requisição da aplicação abrir uma nova conexão, executar uma query e depois fechar a conexão, o sistema pode ter um overhead muito alto.

Em cenários com muitas requisições simultâneas, isso pode causar gargalo no banco.

A ideia do **connection pooling** é manter um conjunto de conexões abertas e reutilizá-las entre várias requisições.

Fluxo sem pool:

```txt
Request -> abre conexão -> executa query -> fecha conexão
Request -> abre conexão -> executa query -> fecha conexão
Request -> abre conexão -> executa query -> fecha conexão
```

Fluxo com pool:

```txt
Request -> pega conexão do pool -> executa query -> devolve conexão ao pool
```

Isso reduz o custo de abrir e fechar conexões repetidamente.

Connection pooling é especialmente importante em aplicações web com muitas requisições concorrentes e também em ambientes serverless, onde várias instâncias podem abrir conexões ao mesmo tempo.

Exemplos de soluções relacionadas:

* PgBouncer;
* RDS Proxy;
* Prisma Accelerate;
* Supabase Pooler;
* pools nativos de ORMs e drivers.

Trade-offs:

* pool mal configurado pode esgotar conexões;
* muitas instâncias da aplicação podem criar pools demais;
* um pool muito pequeno pode virar gargalo;
* um pool muito grande pode sobrecarregar o banco.

---

Índices e connection pooling resolvem gargalos diferentes e costumam ser otimizações iniciais importantes. O ganho real depende das consultas, do hardware, da concorrência e da configuração; não existe um número de requisições por segundo garantido para todos os sistemas.

![indice-connection-polling.png](../assets/indice-connection-polling.png)

A imagem ilustra um cenário em que a combinação reduziu bastante o tempo de resposta, mas esse resultado não deve ser generalizado sem medição.
