# Message Queue

Uma fila serve para a gente armazenar mensagens que serão processadas posteriormente. Uma Message Queue ou fila de mensagens é uma estrutura de dados que permite a comunicação assíncrona entre processos, normalmente a gente vai ser um servidor que se comunica com uma fila de mensagens. Normalmente elas são:

- Desacopladas, então o servidor não está alocado no mesmo lugar que a fila e não precisa saber quem está consumindo as mensagens
- Tem um produtor e consumidor, o produtor envia as mensagens e o consumidor as processa, normalmente dividos em diferentes servidores, porém podem sim estar no mesmo lugar.
- Pode ser FIFO (First In, First Out), isso depende da necessidade da aplicação.
- Buffer que armazena as mensagens por tempo indeterminado, até que sejam processadas com o objetivo de estabilizar a carga de requisições, então em horários de pico a fila vai ficar mais cheia porque o servidor enviar uma quantidade de mensagens que o consumidor não consegue processar imediatamente, porém em algum momento do dia onde essa carga de requests diminui, a fila vai ficar menos cheia porque o consumidor vai conseguir processar as mensagens, então o buffer vai ficar vazio, ou seja, ele serve para essa estabilização da carga de requisições.
- Fan out ou dispersão - Quando o produtor envia uma mensagem, ela é replicada para todas as filas associadas, permitindo que várias instâncias de consumidores processem a mensagem simultaneamente e fazendo com que caso um processo falhe, a mensagem não seja perdida e possa ser processada por outro consumidor.
- Isolação - Quando uma fila falha, as mensagens não são perdidas e podem ser processadas por outras filas, garantindo a continuidade do serviço.
- Disponibilidade - Quando uma fila falha, as mensagens não são perdidas e podem ser processadas por outras filas, garantindo a continuidade do serviço.
- É assíncrono - As mensagens são processadas de forma assíncrona, ou seja, o produtor não espera que a mensagem seja processada antes de continuar enviando outras mensagens.

## Trade-offs

Filas não são para aumentar a performace de um request que meu usuário está fazendo, elas são para garantir a continuidade do serviço mesmo quando ocorrem falhas. Inclusive normalmente filas deixam o processo mais lento, pois o consumidor precisa esperar que a mensagem seja processada antes de continuar. 

Esse é o principal trade-off de filas onde elas não são uma boa caso você precisa de Cônsistencia forte e imetiata e baixa latência. Outro trade-off é a complexidade adicional de gerenciar filas e consumidores e pode deixar o serviço mais lento.

## Delivery semantics / Semântica de Entrega

- At-most-once - A mensagem é entregue no máximo uma vez, ou seja, pode ser perdida se o consumidor falhar antes de processar a mensagem. Muito pouco usado.
- At-least-once - A mensagem é entregue pelo menos uma vez, ou seja, não pode ser perdida se o consumidor falhar antes de processar a mensagem. Essa é a semântica padrão de entrega e a mais usada.
- At-exactly-once - A mensagem é entregue exatamente uma vez, ou seja, não pode ser perdida e não pode ser processada mais de uma vez. Usado em sistemas de transações financeiras e de e-commerce porém com um custo maior de implementação e maior complexidade.

## Dead Letter Queue

Uma fila especial que é usada para armazenar mensagens que não puderam ser processadas corretamente, permitindo que elas sejam analisadas e corrigidas posteriormente. A ideia e bem simples, teve um item na fila que falhou e não puder ser processado, então ele é enviado para a DLQ e com isso eu posso fazer o tratamento necessário para corrigir a mensagem.
