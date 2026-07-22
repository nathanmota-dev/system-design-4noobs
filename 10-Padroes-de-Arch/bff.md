# Backend for Frontend (BFF)

BFF é um padrão de arquitetura que cria uma camada de backend específica para cada tipo de frontend. Ele é útil quando um mesmo sistema possui múltiplos clientes, como uma aplicação web e um aplicativo mobile, que têm necessidades diferentes de dados e de interação.

Imagine uma tela de dashboard que precisa fazer cinco requisições quando é aberta. No aplicativo mobile, essa mesma tela pode precisar de seis requisições, com dados e formatos diferentes. Em vez de o frontend chamar vários serviços diretamente, o BFF expõe um endpoint adaptado às necessidades daquele cliente e coordena as chamadas aos serviços internos.

Embora o resultado possa ser parecido com o de uma API GraphQL, os conceitos são diferentes: BFF é um padrão de arquitetura, enquanto GraphQL é uma linguagem de consulta e um modelo para criação de APIs. Eles podem ser usados separadamente ou em conjunto.

O BFF reduz a quantidade de chamadas entre o cliente e o backend e pode agregar, filtrar, transformar ou buscar dados em paralelo. Isso não significa necessariamente que o backend fará menos chamadas aos serviços ou ao banco de dados: esse ganho depende de estratégias como paralelismo, cache e *batching*.

O principal trade-off é transferir parte da lógica de composição do frontend para o backend. Com isso, o frontend fica mais simples, mas o sistema ganha um componente adicional, que precisa ser mantido, monitorado e escalado. Também pode haver duplicação de lógica quando diferentes BFFs precisam das mesmas regras.

Não é necessário criar um BFF para todo serviço. O padrão costuma fazer mais sentido quando existem vários frontends, quando cada cliente exige uma experiência diferente ou quando a composição de dados no cliente gera muitas chamadas e lógica de integração.
