# NoSQL

Essa nota complementa o arquivo `SQLxNoSql.md`. A ideia aqui nao e aprofundar teoria, e sim passar rapidamente pelos principais tipos de bancos NoSQL, citar nomes comuns e mostrar onde eles costumam aparecer em aplicacoes de system design.

## Quando NoSQL costuma entrar

- quando o schema muda com frequencia
- quando o dado nao encaixa bem em tabelas com muitos joins
- quando precisa escalar horizontalmente com facilidade
- quando leitura e escrita por chave sao mais importantes que consultas relacionais complexas
- quando parte do sistema aceita consistencia eventual em troca de disponibilidade e throughput

NoSQL nao significa "melhor que SQL". Normalmente significa que o problema pede outro tipo de modelagem.

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

- cache
- sessao de usuario
- rate limiting
- feature flags
- carrinho temporario
- contadores e leaderboards

Quando pensar nisso em system design:

- quando voce sabe exatamente a chave que vai buscar
- quando precisa de latencia muito baixa
- quando o volume de leitura e escrita e alto

Observacao:

- Redis aparece muito como cache e armazenamento de dados temporarios
- DynamoDB aparece bastante em sistemas distribuidos na AWS

### 2. Document Store

Exemplos: MongoDB, CouchDB, Firestore

Modelo:

```json
{
  "user_id": 123,
  "name": "John",
  "address": {
    "city": "Sao Paulo",
    "zip": "01000-000"
  },
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

Casos de uso comuns:

- perfil de usuario
- catalogo de produtos
- configuracoes por cliente
- CMS
- MVPs com schema mudando rapido

Quando pensar nisso em system design:

- quando os dados sao naturalmente um "documento"
- quando o objeto costuma ser lido e salvo quase inteiro
- quando o schema precisa evoluir sem muita friccao

Observacao:

- MongoDB e uma escolha comum quando o time quer flexibilidade e modelagem proxima de JSON
- funciona bem quando o agregado do dominio cabe bem dentro de um documento

### 3. Graph DB

Exemplos: Neo4j, ArangoDB, Amazon Neptune

Modelo:

- nodes e edges
- o relacionamento e tao importante quanto o dado

Casos de uso comuns:

- rede social
- recomendacao baseada em conexoes
- deteccao de fraude
- mapa de dependencias entre servicos
- permissionamento complexo

Quando pensar nisso em system design:

- quando a pergunta principal e "como A se relaciona com B?"
- quando consultas de caminho e vizinhanca sao frequentes

Observacao:

- graph DB normalmente nao e o banco principal da aplicacao inteira
- ele brilha quando relacao e navegacao entre entidades sao o centro do problema

### 4. Wide-Column / Column-Family

Exemplos: Cassandra, HBase, ScyllaDB

Modelo:

- dados distribuidos por chave
- colunas agrupadas em familias
- modelagem muito orientada para as queries que o sistema precisa responder

Casos de uso comuns:

- eventos
- telemetria
- series temporais
- historico muito grande
- cargas com muitas escritas

Quando pensar nisso em system design:

- quando precisa de alto throughput de escrita
- quando precisa distribuir dados em varios nos ou varias regioes
- quando disponibilidade e escala horizontal sao prioridade

Observacao:

- Cassandra aparece bastante em cenarios write-heavy
- nao e banco para joins complexos ou consultas ad hoc ricas

### 5. Vector DB

Exemplos: Pinecone, Weaviate, Milvus, Chroma

Modelo:

- armazena embeddings
- busca por similaridade entre vetores

Casos de uso comuns:

- busca semantica
- RAG
- recomendacao por similaridade
- clustering de conteudo
- deduplicacao aproximada

Quando pensar nisso em system design:

- quando voce quer encontrar "itens parecidos" e nao apenas "itens iguais"
- quando o sistema usa IA para recuperar contexto

Observacao:

- vector DB normalmente complementa outro banco
- ele nao substitui um banco transacional da aplicacao

## Resumo rapido

- Redis: cache, sessao, contador, rate limit
- MongoDB: documentos flexiveis, catalogos, perfis, configs
- Neo4j: relacoes complexas e navegacao entre entidades
- Cassandra: alto volume de escrita e distribuicao
- Pinecone/Weaviate/Chroma: busca semantica e IA

## Regra pratica

Em system design, o raciocinio costuma ser:

- se o problema principal e relacionamento forte e integridade transacional, pense primeiro em SQL
- se o problema principal e escala, flexibilidade de schema ou acesso muito especifico ao dado, algum NoSQL pode fazer mais sentido

Na pratica, muitos sistemas usam os dois. Exemplo comum:

- PostgreSQL para dado transacional
- Redis para cache
- MongoDB para documentos flexiveis

Ou seja: NoSQL raramente entra sozinho; ele costuma entrar para resolver uma parte especifica do sistema muito bem.
