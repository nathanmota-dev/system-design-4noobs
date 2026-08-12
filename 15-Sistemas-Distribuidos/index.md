# Sistemas Distribuídos

Um sistema distribuído é formado por componentes independentes que se comunicam por uma rede para oferecer uma capacidade conjunta. Esses componentes podem ser serviços, bancos de dados, filas, caches, workers ou instâncias espalhadas por diferentes máquinas, zonas e regiões.

A rede é o que permite escalar e separar responsabilidades, mas também introduz incerteza. Uma mensagem pode atrasar, ser perdida, chegar duplicada ou chegar fora de ordem. Um componente pode estar funcionando enquanto outro está inacessível. Por isso, System Design exige raciocinar sobre falhas parciais, consistência, coordenação e limites de escala.

## Por que distribuir um sistema?

As principais motivações são:

- aumentar a capacidade de leitura e escrita;
- distribuir tráfego entre várias instâncias;
- isolar domínios e equipes;
- reduzir latência aproximando dados dos usuários;
- aumentar a disponibilidade com redundância;
- processar tarefas em paralelo;
- lidar com volumes que não cabem em um único nó.

Distribuir não é automaticamente melhor. Um monólito bem estruturado pode ser mais simples de operar e suficiente para muitos cenários. A distribuição deve resolver uma necessidade real de escala, disponibilidade, isolamento ou organização.

## O que a rede muda?

Quando dois componentes estão na mesma função ou processo, uma chamada pode ser direta e previsível. Quando estão separados por uma rede, surgem novos problemas:

- latência variável;
- timeout;
- falha parcial;
- perda ou duplicidade de mensagens;
- quebra de conexão depois do processamento;
- incompatibilidade de versão;
- sobrecarga de serialização e transporte;
- necessidade de autenticação entre serviços.

Uma resposta que não chegou não permite concluir, sozinha, se a operação não foi executada. O serviço pode ter processado o pedido e falhado antes de enviar a resposta. Esse cenário é uma das razões para usar idempotência e identificadores de operação.

## Falhas parciais

Em um sistema distribuído, não existe apenas o estado “tudo funcionando” ou “tudo parado”. É possível que:

- o serviço A esteja saudável e o serviço B esteja lento;
- a réplica esteja viva, mas atrasada;
- uma região consiga falar com outra apenas em uma direção;
- o produtor publique eventos e o consumidor esteja indisponível;
- metade dos nós veja uma configuração diferente da outra metade.

O design precisa definir como responder nesses estados intermediários. Timeout, circuit breaker, quorum, retry limitado e degradação controlada são exemplos de mecanismos usados para lidar com essas situações.

## Consistência

Consistência descreve o quanto diferentes leitores observam uma visão alinhada dos dados.

### Consistência forte

Depois que uma escrita é confirmada, as leituras posteriores observam o novo valor, respeitando as garantias do sistema.

É útil para operações como saldo, autorização, inventário limitado ou regras que não podem aceitar um estado antigo. O custo pode ser maior latência e menor disponibilidade durante falhas de comunicação.

### Consistência eventual

As réplicas podem divergir temporariamente, mas convergem quando as atualizações são propagadas.

É comum em feeds, contadores aproximados, catálogos, índices de busca e dados replicados globalmente. O sistema ganha disponibilidade e pode reduzir latência, mas o usuário pode observar um valor antigo por algum tempo.

### Consistência causal

Preserva a relação entre eventos que dependem uns dos outros. Se uma operação B depende de A, os leitores não deveriam observar B antes de A.

Esse modelo fica entre garantias fortes e eventualidade simples, mas aumenta a complexidade de rastrear dependências e versões.

A escolha depende do domínio. “Eventual” não deve ser usado apenas porque é mais escalável; é necessário explicar por que o negócio tolera o atraso.

## Teorema CAP

O teorema CAP trata de sistemas distribuídos diante de uma partição de rede. Durante uma partição, o sistema precisa escolher entre:

- **Consistência:** preservar uma visão coordenada dos dados;
- **Disponibilidade:** continuar respondendo a requisições;
- **Tolerância a partições:** continuar operando mesmo com a comunicação entre nós interrompida.

A tolerância a partições não é opcional em uma rede real. Portanto, quando ocorre uma partição, o sistema precisa priorizar consistência ou disponibilidade para aquela operação.

Isso não significa que todo sistema seja simplesmente “CP” ou “AP” em qualquer situação. Componentes e operações diferentes podem fazer escolhas diferentes. Um cadastro de usuário pode priorizar consistência, enquanto um contador de visualizações pode priorizar disponibilidade.

O repositório também possui uma explicação específica em [Teorema CAP](../01-BD/04-06-Teorema-CAP.md).

## Replicação

Replicação mantém cópias dos dados em mais de um nó.

### Líder e seguidores

Um líder recebe escritas e replica as alterações para seguidores. Leituras podem ser direcionadas ao líder ou às réplicas.

- **Vantagens:** modelo de escrita simples, ordenação centralizada e boa distribuição de leitura.
- **Desvantagens:** o líder pode ser gargalo; failover exige eleição; réplicas assíncronas podem estar atrasadas.

### Multi-leader

Mais de um nó aceita escrita. É útil para múltiplas regiões, mas exige resolver conflitos e definir regras de ordenação.

- **Vantagens:** menor latência de escrita local e maior disponibilidade regional.
- **Desvantagens:** conflitos, resolução de versões, maior complexidade e risco de divergência.

### Replicação síncrona e assíncrona

- **Síncrona:** a escrita só é confirmada depois de chegar a um conjunto de réplicas. Reduz perda de dados, mas aumenta latência e pode bloquear durante falhas.
- **Assíncrona:** o líder confirma antes de todas as réplicas receberem. Tem menor latência, mas pode perder as últimas escritas se o líder falhar.

Não existe uma configuração universalmente melhor. A decisão depende de RPO, RTO, latência e criticidade dos dados.

## Quorum

Em alguns sistemas, uma operação precisa ser confirmada por uma quantidade mínima de nós.

Se existem `N` réplicas, `W` confirmações para escrita e `R` réplicas consultadas para leitura, escolher `W + R > N` aumenta a chance de a leitura encontrar uma cópia atualizada. Essa relação não substitui a análise das garantias concretas da tecnologia, mas ajuda a raciocinar sobre o desenho.

- `W` alto aumenta a durabilidade percebida, mas torna a escrita mais sensível a falhas.
- `R` alto aumenta a chance de encontrar o valor mais recente, mas aumenta custo e latência da leitura.
- valores baixos melhoram disponibilidade e desempenho, mas podem retornar dados antigos.

## Particionamento e sharding

Quando um único nó não suporta o volume de dados ou tráfego, os dados podem ser divididos entre partições.

A chave de partição deve:

- distribuir a carga de forma equilibrada;
- permitir encontrar o dado sem consultar todos os nós;
- preservar a proximidade de dados que são lidos juntos, quando necessário;
- evitar que uma chave muito popular vire hot key.

Uma escolha ruim pode concentrar grande parte das requisições em um único nó. Alterar a chave depois também pode exigir rebalanceamento e migração de dados.

Mais detalhes estão em [Sharding e Partitioning](../01-BD/04-05-Sharding-e-Partitioning.md).

## Ordem e tempo

Máquinas diferentes não compartilham um relógio perfeitamente sincronizado. Timestamps podem ter pequenas diferenças e não devem ser usados sozinhos para garantir a ordem de eventos.

Dependendo do caso, a ordem pode ser definida por:

- número de sequência por entidade;
- offset de uma partição;
- versão esperada do registro;
- relógio lógico;
- líder responsável por ordenar operações.

Também é preciso definir o escopo da ordem. Ordem global é cara e difícil de escalar. Muitos sistemas garantem ordem apenas por usuário, pedido, conta ou partição.

## Coordenação e consenso

Alguns problemas exigem que nós concordem sobre uma decisão, como:

- qual nó é o líder;
- qual configuração está ativa;
- qual operação foi confirmada;
- qual sequência de eventos deve ser seguida.

Algoritmos de consenso, como Raft e Paxos, existem para lidar com esse tipo de coordenação. Na prática, normalmente é melhor usar um componente ou serviço maduro do que implementar consenso dentro da aplicação.

Coordenação aumenta as garantias, mas pode aumentar latência, reduzir disponibilidade durante partições e criar dependências operacionais. Uma aplicação deve evitar coordenação global quando o problema pode ser resolvido com particionamento, processamento assíncrono ou consistência local.

## Transações distribuídas

Uma transação local pode ser atômica dentro de um banco. Quando uma operação envolve vários bancos ou serviços, manter uma transação única é mais difícil.

Algumas alternativas:

- **Two-Phase Commit:** coordena participantes em duas fases. Pode fornecer garantias fortes, mas mantém recursos bloqueados e é sensível à falha do coordenador.
- **Saga:** divide o fluxo em etapas e define ações de compensação. É mais flexível e escalável, mas produz consistência eventual e exige lidar com estados intermediários.
- **Outbox e eventos:** registra a mudança e a intenção de publicar no mesmo banco local, permitindo que um processo separado publique o evento com segurança.

O padrão adequado depende de o negócio aceitar compensação, atraso e estados intermediários. [Saga](../10-Padroes-de-Arch/saga.md) e [Event-Driven](../10-Padroes-de-Arch/event-driven.md) aprofundam esses padrões.

## Idempotência e mensagens

Comunicação distribuída frequentemente usa entrega `at-least-once`, o que significa que uma mensagem pode ser processada mais de uma vez. O consumidor precisa evitar efeitos duplicados usando:

- chave idempotente;
- tabela de mensagens processadas;
- versão do recurso;
- operações naturalmente idempotentes;
- deduplicação com retenção adequada.

Também é necessário pensar em mensagens fora de ordem, eventos atrasados e eventos que não podem mais ser processados porque o schema mudou.

## Exemplo: criação de pedido

Considere um sistema com serviços de pedidos, estoque, pagamento e notificações:

```text
Cliente → Order Service → Banco de Pedidos
                    ├── evento → Estoque
                    ├── evento → Pagamento
                    └── evento → Notificação
```

Uma possível decisão é manter a criação do pedido e sua chave idempotente em uma transação local. Depois, um evento é publicado por Outbox. Os consumidores processam o evento de forma idempotente e atualizam seus próprios estados.

Essa arquitetura escala e desacopla os serviços, mas o cliente pode consultar o pedido antes de estoque ou pagamento terem terminado. A API precisa comunicar esse estado e o domínio precisa definir o que acontece quando uma etapa falha.

## Trade-offs

- **Distribuição x simplicidade:** mais nós aumentam capacidade e isolamento, mas tornam falhas e diagnósticos mais complexos.
- **Consistência x disponibilidade:** garantias fortes podem bloquear durante falhas; disponibilidade pode expor dados antigos ou conflitos.
- **Replicação síncrona x latência:** durabilidade maior exige esperar mais participantes.
- **Replicação assíncrona x perda de dados:** menor latência pode deixar alterações recentes vulneráveis durante um failover.
- **Ordem global x escala:** ordenar tudo simplifica alguns consumidores, mas cria coordenação e gargalo.
- **Transação distribuída x desacoplamento:** coordenação forte facilita certas invariantes, mas reduz autonomia e disponibilidade.
- **Multi-região x operação:** aproxima o sistema dos usuários, mas exige resolver conflitos, failover, dados e observabilidade entre regiões.

O objetivo não é eliminar toda inconsistência ou toda falha. É escolher garantias que façam sentido para cada parte do domínio.

## Como aplicar em uma entrevista de System Design

Perguntas importantes:

1. Por que esse componente precisa ser distribuído?
2. Qual falha acontece se a rede entre dois nós for interrompida?
3. Que nível de consistência cada operação exige?
4. Como as réplicas convergem ou resolvem conflitos?
5. Qual é a chave de particionamento e como evitar hot keys?
6. Como tratar duplicidade, reordenação e mensagens atrasadas?
7. Existe algum ponto que exige consenso ou ordem global?
8. Qual é o comportamento durante o failover?

## Resumo

Sistemas distribuídos permitem escalar, aumentar disponibilidade e reduzir latência, mas introduzem falhas parciais, atrasos, duplicidade, conflitos e necessidade de coordenação. Consistência, replicação, particionamento, consenso, transações e idempotência são partes do mesmo problema: decidir quais garantias o sistema oferece quando seus componentes não conseguem agir como uma única máquina.

Em uma entrevista de System Design, a resposta fica mais forte quando explica por que distribuir, quais garantias são necessárias e quais trade-offs serão aceitos.
