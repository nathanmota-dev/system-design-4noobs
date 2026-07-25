# Padrão Saga

Saga é um padrão para coordenar uma operação de negócio que atravessa vários serviços ou sistemas. Em vez de manter uma única transação distribuída, a saga divide o fluxo em uma sequência de transações locais. Cada serviço confirma sua própria transação; se uma etapa posterior falhar, o sistema executa ações de compensação para desfazer ou neutralizar as etapas que já foram concluídas.

## O problema

Imagine uma agência de viagens que precisa reservar:

- uma passagem aérea;
- um hotel;
- um carro.

Cada recurso pode pertencer a um serviço ou até a um fornecedor diferente. Manter todos os bancos bloqueados até que as três reservas terminem pode aumentar o tempo das transações, manter recursos indisponíveis e exigir que todos os participantes estejam disponíveis ao mesmo tempo.

Uma alternativa é o Two-Phase Commit (2PC), que tenta coordenar uma transação distribuída com fases de preparação e confirmação. Ele pode oferecer uma semântica mais próxima de atomicidade, mas exige coordenação forte, aumenta o tempo de retenção de recursos e não é uma solução simples para fornecedores externos.

Saga é outra alternativa quando o negócio aceita consistência eventual e estados intermediários, como `PENDING` ou `COMPENSATING`.

## Exemplo com orquestração

Um orquestrador mantém o estado da saga e envia comandos aos serviços envolvidos:

1. O cliente solicita a criação da viagem.
2. O orquestrador cria a saga com status `PENDING`.
3. O serviço de voos, o serviço de hotéis e o serviço de carros executam suas transações locais de reserva. Essas etapas podem ser sequenciais ou paralelas.
4. Cada serviço responde com sucesso, falha ou timeout.
5. Se todas as etapas obrigatórias forem concluídas, o orquestrador marca a viagem como `CONFIRMED`.
6. Se uma etapa falhar, o orquestrador envia comandos de compensação para as etapas concluídas, como `CancelFlight` e `CancelHotel`, e marca a saga como `COMPENSATING` ou `FAILED`.

O pedido não deve ser confirmado antes de todas as etapas obrigatórias terminarem. É possível reservar os recursos em paralelo e confirmar cada reserva localmente, mas o estado geral da viagem continua pendente até que o critério de sucesso seja atendido.

As compensações não são um rollback perfeito. Cancelar uma passagem pode gerar multa, o fornecedor pode estar indisponível e algumas ações não têm um inverso exato. Por isso, uma saga precisa lidar com retries, timeouts, idempotência, expiração de reservas e reconciliação manual ou automática.

O estado do orquestrador deve ser persistido. Se ele cair, outra instância só conseguirá continuar do ponto correto se tiver acesso ao estado durável da saga e se as operações puderem ser repetidas com segurança. A recuperação não acontece automaticamente apenas porque eventos foram emitidos.

## Coreografia

Na coreografia, não existe um orquestrador central decidindo cada passo. Os próprios serviços publicam eventos e reagem aos eventos dos demais.

Um fluxo possível seria:

1. O serviço de viagens publica `TripRequested`.
2. Os serviços de voos, hotéis e carros consomem o evento e tentam fazer suas reservas.
3. Cada serviço publica `FlightReserved`, `HotelReserved`, `CarRejected` ou eventos equivalentes.
4. Ao receber uma rejeição, os serviços que já reservaram recursos publicam ou executam suas compensações.

A coreografia reduz a dependência de um coordenador central, mas o fluxo pode ficar difícil de visualizar e depurar. Muitos serviços reagindo uns aos outros podem criar cadeias de eventos, dependências ocultas e ciclos.

## Orquestração x coreografia

### Orquestração

- O fluxo e o estado ficam concentrados em um coordenador.
- É mais fácil observar, testar e alterar a sequência da saga.
- O orquestrador é outro componente para operar e precisa ser altamente disponível.
- O coordenador conhece os participantes e suas operações de compensação.

### Coreografia

- Os serviços são mais independentes e reagem a eventos.
- Não existe um coordenador central para virar gargalo ou ponto único de falha.
- O fluxo distribuído pode ser mais difícil de entender, monitorar e modificar.
- É mais adequada para fluxos relativamente simples; em processos longos, a falta de uma visão central pode se tornar um problema.

## Quando usar

Saga é útil para workflows longos que atravessam microsserviços, bancos diferentes ou APIs externas, quando não é viável manter uma transação distribuída e o negócio aceita consistência eventual.

Se todas as alterações pertencem ao mesmo banco e podem ser resolvidas com uma transação local, essa opção geralmente é mais simples. Se a regra exige atomicidade forte e todos os participantes suportam o protocolo, 2PC pode ser considerado, embora tenha seus próprios custos e limitações.

Em uma entrevista, os pontos essenciais são: transações locais, compensações, consistência eventual, idempotência, retries, timeouts e a escolha entre orquestração e coreografia.
