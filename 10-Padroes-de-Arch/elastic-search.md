# Elasticsearch

Elasticsearch é um mecanismo distribuído de busca e análise que armazena dados em documentos e cria índices para consultá-los com baixa latência. Ele é especialmente útil para busca textual, filtros, ordenação por relevância, autocomplete e agregações.

Em uma arquitetura comum, o banco de dados relacional ou NoSQL continua sendo a fonte principal dos dados. O Elasticsearch recebe uma cópia otimizada para busca. Isso evita colocar consultas textuais complexas no banco transacional, mas também significa que os dados podem ter um pequeno atraso até serem indexados.

## Índice invertido

Um índice invertido mapeia termos para os documentos que os contêm. Em vez de examinar todos os documentos a cada consulta, o mecanismo consulta estruturas previamente construídas. Essa pré-computação melhora a busca, mas consome espaço e torna as operações de indexação e atualização mais caras.

Isso não significa que um banco relacional seja sempre lento para buscar texto: para volumes menores e consultas simples, um índice no próprio banco pode ser suficiente. Um mecanismo de busca passa a ser interessante quando a aplicação precisa de relevância textual, tolerância a variações, filtros combinados, autocomplete ou grande volume de documentos.

## Exemplo

Uma busca por produtos poderia ser feita assim:

```http
GET /products/_search
{
  "query": {
    "match": {
      "name": "tênis"
    }
  },
  "size": 20
}
```

O `match` é apropriado para busca textual analisada. Para filtros exatos, como categoria, status ou identificador, normalmente são usados campos e consultas próprias para valores exatos.

## Fluxo típico

1. A aplicação grava a alteração no banco principal.
2. Um evento, uma fila, CDC ou outro processo de sincronização envia a mudança para o Elasticsearch.
3. O Elasticsearch indexa o documento.
4. As consultas de busca são atendidas pelo índice, e a aplicação pode consultar o banco principal para obter detalhes que exigem consistência forte.

É importante tratar falhas nesse fluxo. O consumidor precisa ser idempotente, e a arquitetura deve ter retries, monitoramento e uma estratégia para reindexar documentos quando necessário.

## Casos de uso

- Catálogos e busca de produtos em e-commerce.
- Busca em documentos, artigos e bases de conhecimento.
- Autocomplete, sugestões e correção de termos.
- Logs, observabilidade e análise de eventos.
- Busca geográfica e filtros por localização.
- RAG (*Retrieval-Augmented Generation*).

Em RAG, o Elasticsearch pode recuperar trechos relevantes por texto, vetores ou uma combinação dos dois. A aplicação então envia esses trechos para um modelo de linguagem gerar a resposta; o Elasticsearch faz a recuperação, não a geração.

## Vantagens

- Busca textual com relevância e análise de linguagem.
- Filtros, ordenação e agregações eficientes.
- Escala horizontal por meio de shards e réplicas.
- Integração com pipelines de logs, eventos e dados derivados.

## Trade-offs

- **Consistência:** a indexação costuma ser *near real-time*, não necessariamente imediata.
- **Duplicação:** os dados ficam no banco principal e no índice, aumentando armazenamento e a responsabilidade de sincronização.
- **Operação:** shards, réplicas, mappings, retenção e capacidade precisam ser monitorados.
- **Transações:** não é a escolha padrão para regras relacionais e transações de negócio que exigem integridade forte.
- **Custo:** um serviço gerenciado pode ser pago, e uma instalação própria também tem custo de infraestrutura e operação.

Elasticsearch e OpenSearch têm funcionalidades e APIs parecidas, mas não são a mesma solução nem possuem compatibilidade perfeita. OpenSearch é uma alternativa de código aberto; a escolha deve considerar recursos necessários, ecossistema, hospedagem, licenciamento e custo total. O fato de o software ser aberto não elimina o custo de executar e operar o serviço.
