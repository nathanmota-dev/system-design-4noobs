# 4. Replicação de leitura

Em muitas aplicações, o volume de leitura é muito maior do que o volume de escrita. Exemplo: em uma rede social, uma pessoa publica um post, mas centenas ou milhares de pessoas podem ler esse mesmo post. Nesse tipo de cenário, podemos usar **read replicas**.

![Comparacao](../assets/replys.png)

A ideia é ter:

* um banco principal para escrita;
* uma ou mais réplicas para leitura.

Exemplo:

```txt
Writes -> Primary DB
Reads  -> Read Replicas
```

Isso ajuda a distribuir a carga e evita que todas as leituras e escritas concorram pelo mesmo banco.

## Exemplo prático

Em uma aplicação como Twitter/X ou LinkedIn:

```txt
1 usuário cria um post.
100 usuários leem esse post.
```

Nesse caso, existe muito mais leitura do que escrita. Então faz sentido escalar leitura com réplicas.

## Trade-off: replication lag

O principal problema é o atraso de replicação.

Quando um dado é escrito no banco principal, ele pode demorar alguns segundos para aparecer nas réplicas.

Exemplo:

```txt
Usuário cria um post -> dado salvo no primary
Réplica ainda não recebeu o dado
Outro usuário pode demorar alguns segundos para ver o post
```

Para redes sociais, isso geralmente é aceitável.

Mas em sistemas bancários, financeiros ou de estoque crítico, esse atraso pode ser problemático.

Nesses casos, é comum:

* ler diretamente do banco principal após uma escrita;
* usar consistência forte em operações críticas;
* evitar réplicas para dados sensíveis;
* aceitar mais latência para garantir consistência.

---
