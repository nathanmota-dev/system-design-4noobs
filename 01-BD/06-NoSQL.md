# NoSQL

Essa nota complementa o arquivo [`02-SQLxNoSql.md`](./02-SQLxNoSql.md). A ideia aqui não é aprofundar a teoria, e sim passar rapidamente pelos principais tipos de bancos NoSQL, citar nomes comuns e mostrar onde eles costumam aparecer em aplicações de System Design.

## Quando NoSQL costuma entrar

- Quando o schema muda com frequência.
- Quando o dado não se encaixa bem em tabelas com muitos joins.
- Quando é preciso escalar horizontalmente com facilidade.
- Quando leitura e escrita por chave são mais importantes do que consultas relacionais complexas.
- Quando parte do sistema aceita consistência eventual em troca de disponibilidade e throughput.

NoSQL não significa "melhor que SQL". Normalmente significa que o problema pede outro tipo de modelagem.

## Principais tipos

### 1. Key-Value

Exemplos: Redis, DynamoDB, Riak

Modelo:

```json
{
  "user:123": {
    "is_premium": true,
    "plan": "gold"
  }
}
```

Casos de uso comuns:

- Cache.
- Sessão de usuário.
- Rate limiting.
- Feature flags.
- Carrinho temporário.
- Contadores e leaderboards.

Quando pensar nisso em System Design:

- Quando você sabe exatamente qual chave vai buscar.
- Quando precisa de latência muito baixa.
- Quando o volume de leitura e escrita é alto.

Observação:

- Redis aparece muito como cache e armazenamento de dados temporários.
- DynamoDB aparece bastante em sistemas distribuídos na AWS.

### 2. Document Store

Exemplos: MongoDB, CouchDB, Firestore

Modelo:

```json
{
  "user_id": 123,
  "name": "John",
  "address": {
    "city": "São Paulo",
    "zip": "01000-000"
  },
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

Casos de uso comuns:

- Perfil de usuário.
- Catálogo de produtos.
- Configurações por cliente.
- CMS.
- MVPs com schema mudando rápido.

Quando pensar nisso em System Design:

- Quando os dados são naturalmente um "documento".
- Quando o objeto costuma ser lido e salvo quase inteiro.
- Quando o schema precisa evoluir sem muita fricção.

Observação:

- MongoDB é uma escolha comum quando o time quer flexibilidade e modelagem próxima de JSON.
- Funciona bem quando o agregado do domínio cabe dentro de um documento.

### 3. Graph DB

Exemplos: Neo4j, ArangoDB, Amazon Neptune

Modelo:

- Nós (*nodes*) e arestas (*edges*).
- O relacionamento é tão importante quanto o dado.

Casos de uso comuns:

- Rede social.
- Recomendação baseada em conexões.
- Detecção de fraude.
- Mapa de dependências entre serviços.
- Controle de permissões complexo.

Quando pensar nisso em System Design:

- Quando a pergunta principal é "como A se relaciona com B?".
- Quando consultas de caminho e vizinhança são frequentes.

Observação:

- Um graph DB normalmente não é o banco principal da aplicação inteira.
- Ele se destaca quando relação e navegação entre entidades são o centro do problema.

### 4. Wide-Column / Column-Family

Exemplos: Cassandra, HBase, ScyllaDB

Modelo:

- Dados distribuídos por chave.
- Colunas agrupadas em famílias.
- Modelagem muito orientada às queries que o sistema precisa responder.

Casos de uso comuns:

- eventos
- telemetria
- Séries temporais.
- Histórico muito grande.
- cargas com muitas escritas

Quando pensar nisso em System Design:

- Quando precisa de alto throughput de escrita.
- Quando precisa distribuir dados em vários nós ou regiões.
- Quando disponibilidade e escala horizontal são prioridade.

Observação:

- Cassandra aparece bastante em cenários com muitas escritas.
- Não é um banco voltado a joins complexos ou consultas *ad hoc* ricas.

### 5. Vector DB

Exemplos: Pinecone, Weaviate, Milvus, Chroma

Modelo:

- armazena embeddings
- busca por similaridade entre vetores

Casos de uso comuns:

- Busca semântica.
- RAG
- Recomendação por similaridade.
- Agrupamento de conteúdo.
- Deduplicação aproximada.

Quando pensar nisso em System Design:

- Quando você quer encontrar "itens parecidos", e não apenas "itens iguais".
- Quando o sistema usa IA para recuperar contexto.

Observação:

- Um vector DB normalmente complementa outro banco.
- Ele não substitui um banco transacional da aplicação.

## Resumo rápido

- Redis: cache, sessão, contador e rate limit.
- MongoDB: documentos flexíveis, catálogos, perfis e configurações.
- Neo4j: relações complexas e navegação entre entidades.
- Cassandra: alto volume de escrita e distribuição.
- Pinecone/Weaviate/Chroma: busca semântica e IA.

## Regra prática

Em System Design, o raciocínio costuma ser:

- Se o problema principal é relacionamento forte e integridade transacional, pense primeiro em SQL.
- Se o problema principal é escala, flexibilidade de schema ou acesso muito específico ao dado, algum NoSQL pode fazer mais sentido.

Na prática, muitos sistemas usam os dois. Exemplo comum:

- PostgreSQL para dados transacionais.
- Redis para cache.
- MongoDB para documentos flexíveis.

Ou seja: NoSQL raramente entra sozinho; ele costuma entrar para resolver muito bem uma parte específica do sistema.
