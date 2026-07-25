# Arquitetura orientada a eventos (Event-Driven Architecture)

Uma arquitetura orientada a eventos organiza parte da comunicação do sistema em torno de eventos. Um evento é um registro de um fato que já aconteceu, por exemplo:

- O usuário comprou o item `Y`.
- O pagamento `Z` falhou.
- O pedido `G` foi cancelado.
- O item `H` foi devolvido.

Um produtor publica o evento em um broker, event bus ou stream, e um ou mais consumidores reagem a ele:

```text
Produtor -> Broker/Event Bus -> Consumidor de notificações
                           -> Consumidor de analytics
                           -> Consumidor de busca
```

O produtor não precisa conhecer cada consumidor. Porém, isso não elimina os contratos: nome, schema, versão e semântica do evento continuam sendo responsabilidades importantes.

## Evento não é comando

- Um **comando** pede que uma ação seja executada, como `ReserveFlight`.
- Um **evento** informa que uma ação já ocorreu, como `FlightReserved`.

Um evento pode ser consumido por vários componentes, enquanto um comando normalmente tem um responsável específico. Essa distinção ajuda a evitar que um consumidor interprete um fato como uma solicitação duplicada.

## Características importantes

- **Assíncrona na maioria dos casos:** o produtor publica e pode continuar sem aguardar todos os consumidores. Event-Driven não significa que absolutamente todo processamento precise ser assíncrono; alguns handlers podem responder de forma síncrona.
- **Desacoplamento:** produtores e consumidores podem evoluir e escalar separadamente, desde que respeitem o contrato do evento.
- **Imutabilidade semântica:** depois que um fato ocorreu, ele não deve ser alterado para representar outro fato. Se o pedido for cancelado, publica-se um novo evento, como `OrderCancelled`.
- **Retenção por política:** eventos podem ser retidos por um período e depois removidos por custo, privacidade ou conformidade. Imutabilidade não significa armazenamento infinito.
- **Fan-out:** vários consumidores podem reagir ao mesmo evento para finalidades diferentes.

Em muitos sistemas, a entrega é `at-least-once`. Por isso, o mesmo evento pode chegar mais de uma vez, e os consumidores precisam ser idempotentes. A ordem também precisa ser definida: normalmente ela é garantida apenas por entidade, chave ou partição, não globalmente.

## Vantagens

- **Desacoplamento temporal e estrutural:** o produtor não precisa esperar nem conhecer todos os consumidores.
- **Escalabilidade:** cada consumidor pode ser escalado de acordo com sua própria carga.
- **Resiliência:** filas e streams podem absorver picos e permitir retries quando uma dependência está indisponível.
- **Extensibilidade:** novos consumidores, como notificações ou analytics, podem ser adicionados sem alterar o produtor.
- **Processamento assíncrono:** tarefas demoradas podem sair do caminho crítico da requisição do usuário.

## Desvantagens e cuidados

- **Consistência eventual:** uma atualização pode aparecer em um consumidor antes de aparecer em outro.
- **Debug mais difícil:** é necessário rastrear o evento, os retries e cada consumidor envolvido.
- **Duplicidade e ordenação:** consumidores precisam lidar com reentrega, eventos fora de ordem e mensagens atrasadas.
- **Schema e versionamento:** alterações incompatíveis podem quebrar consumidores antigos; é necessário definir compatibilidade e versionar contratos.
- **Operação adicional:** brokers, filas, consumidores, DLQs, monitoramento e alertas aumentam a complexidade.

Também é preciso evitar a inconsistência entre gravar no banco e publicar o evento. Uma alternativa comum é usar o padrão Transactional Outbox ou CDC, para que a mudança e o registro da publicação possam ser recuperados de forma confiável.

## Quando usar

Event-Driven costuma ser uma boa opção para notificações, integração entre serviços, processamento de pedidos, atualização de índices, analytics e tarefas que podem terminar depois da resposta inicial.

Se uma operação precisa validar várias regras e devolver uma resposta imediata e atômica, uma chamada síncrona e uma transação local podem ser mais simples. Em uma entrevista, vale explicar qual parte do fluxo precisa ser síncrona e qual pode aceitar consistência eventual.
