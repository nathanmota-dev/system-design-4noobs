# Resiliência

Resiliência é a capacidade de um sistema continuar funcionando, ainda que de forma degradada, quando componentes, dependências ou redes falham. Em System Design, isso significa projetar o sistema considerando que falhas irão acontecer, em vez de assumir que todos os componentes estarão sempre disponíveis.

Disponibilidade e resiliência estão relacionadas, mas não são a mesma coisa. Disponibilidade mede se o serviço está acessível. Resiliência descreve como ele reage a falhas, como limita o impacto e como se recupera.

## Pensando em falhas

Um sistema distribuído pode falhar de várias formas:

- uma instância pode parar;
- uma zona ou região pode ficar indisponível;
- uma dependência pode responder lentamente;
- uma mensagem pode ser entregue mais de uma vez;
- uma conexão pode ser interrompida depois de uma operação ter sido processada;
- uma fila pode crescer mais rápido do que os consumidores conseguem processar;
- uma configuração ou um deploy pode introduzir um erro;
- uma dependência externa pode retornar respostas inválidas ou sofrer um ataque.

O primeiro passo é definir o que o sistema deve fazer em cada falha. Nem toda operação precisa continuar. Em alguns casos, recusar rapidamente é melhor do que manter requisições presas e consumir todos os recursos.

## Objetivos de resiliência

Antes de escolher padrões, é importante definir:

- qual indisponibilidade é aceitável;
- quais funcionalidades são críticas;
- quais dados podem ser perdidos ou reprocessados;
- quanto tempo o sistema pode levar para se recuperar;
- quanto tráfego deve ser suportado durante uma degradação;
- qual é o limite de latência aceitável;
- se a consistência precisa ser forte ou pode ser eventual.

Essas decisões conectam resiliência aos requisitos do sistema. Um feed social pode continuar mostrando dados armazenados em cache. Um sistema de autorização de pagamento talvez precise bloquear a operação se não conseguir validar o estado correto.

## Timeouts e deadlines

Toda chamada para uma rede ou dependência externa deveria ter timeout. Sem timeout, uma conexão presa pode ocupar threads, conexões e memória até derrubar a aplicação.

Um timeout precisa ser compatível com o orçamento de latência da requisição. Se o cliente aceita 500 ms, não faz sentido um serviço interno esperar 5 segundos por uma dependência.

Em fluxos com várias chamadas, é útil propagar um deadline: o tempo total restante para concluir a operação. Cada serviço usa apenas uma parte desse orçamento e cancela o trabalho quando não há tempo suficiente para produzir uma resposta válida.

### Trade-offs dos timeouts

- Timeout curto libera recursos rapidamente, mas pode cancelar operações que ainda conseguiriam terminar.
- Timeout longo aumenta a chance de sucesso em uma chamada lenta, mas mantém recursos ocupados e aumenta a latência percebida.
- Um timeout diferente por dependência permite maior precisão, mas aumenta a complexidade de configuração.

## Retries

Retry é uma nova tentativa após uma falha temporária, como uma desconexão ou resposta `503`. Ele pode ajudar em falhas transitórias, mas não corrige erros permanentes.

Uma estratégia comum é usar exponential backoff com jitter:

1. a primeira tentativa falha;
2. o cliente espera um pequeno intervalo;
3. o intervalo aumenta a cada tentativa;
4. um componente aleatório evita que muitos clientes tentem novamente ao mesmo tempo;
5. existe um limite de tentativas e de tempo total.

Retries devem ser usados com cuidado em operações que alteram estado. Se a resposta se perder depois de o servidor processar a operação, repetir a chamada pode duplicar o efeito. Nesses casos, idempotência ou uma chave de operação é necessária.

### Retry storm

Quando uma dependência fica lenta, milhares de clientes podem fazer retries simultaneamente. O tráfego adicional piora a falha e pode derrubar a dependência completamente.

Para evitar isso:

- limite o número de tentativas;
- use backoff e jitter;
- aplique timeout;
- interrompa chamadas com circuit breaker;
- limite a concorrência;
- diferencie erros temporários de erros definitivos.

## Circuit breaker

O circuit breaker evita continuar chamando uma dependência que está falhando. Ele costuma ter três estados:

- **Closed:** as chamadas passam normalmente e as falhas são observadas.
- **Open:** as chamadas são rejeitadas rapidamente ou seguem para um fallback.
- **Half-open:** algumas chamadas de teste são liberadas para verificar se a dependência se recuperou.

Isso protege o serviço local contra latência e falhas de uma dependência. Porém, os limites precisam ser calibrados. Um circuito muito sensível pode abrir por uma variação pequena; um circuito permissivo pode reagir tarde demais.

## Bulkhead e isolamento

O padrão bulkhead separa recursos para que uma falha não consuma toda a capacidade do serviço. O isolamento pode ser feito por:

- pools de conexão;
- filas de execução;
- limites de concorrência;
- processos ou containers diferentes;
- separação por cliente, tenant ou tipo de operação;
- workers específicos para tarefas pesadas.

Por exemplo, uma integração lenta com um parceiro externo não deveria ocupar todas as threads usadas para consultar pedidos. O custo é que recursos isolados podem ficar ociosos enquanto outro pool está saturado, mas essa perda de utilização pode ser aceitável para limitar o raio de impacto.

## Rate limiting e backpressure

Rate limiting limita a quantidade de requisições aceitas por cliente, rota ou sistema. Backpressure faz o produtor desacelerar quando o consumidor não consegue acompanhar.

Esses mecanismos protegem o sistema contra picos, abuso e sobrecarga. Podem ser implementados com token bucket, leaky bucket, filas ou limites de concorrência.

O limite precisa considerar a capacidade real. Um limite muito alto não protege; um limite muito baixo rejeita tráfego legítimo. Também é importante definir o que acontece ao atingir o limite: responder `429`, colocar em fila, degradar a resposta ou usar uma prioridade diferente.

## Filas, retries e Dead Letter Queue

Filas desacoplam o produtor do consumidor e absorvem picos. Elas são úteis quando o trabalho pode ser processado de forma assíncrona.

Um consumidor pode tentar novamente uma mensagem que falhou. Depois de um número limitado de tentativas, a mensagem pode ser enviada para uma Dead Letter Queue (DLQ) para investigação ou processamento manual.

Esse desenho traz alguns custos:

- a resposta pode não ser imediata;
- o sistema passa a ter consistência eventual;
- a mensagem pode ser processada mais de uma vez;
- é necessário monitorar idade e tamanho da fila;
- o consumidor precisa ser idempotente.

Mais detalhes sobre filas e entrega de mensagens estão em [Message Queue](../02-Queue-Filas/01-Message-Queue.md).

## Fallback e degradação controlada

Quando uma dependência falha, o sistema pode oferecer uma resposta simplificada:

- usar dados em cache;
- ocultar uma funcionalidade não essencial;
- mostrar o último estado conhecido;
- aceitar a solicitação e processá-la depois;
- retornar uma resposta parcial;
- direcionar o usuário para um fluxo alternativo.

Fallback não significa esconder qualquer erro. É preciso distinguir uma resposta válida, porém degradada, de uma resposta incorreta. Em operações financeiras, por exemplo, usar um saldo antigo como se fosse atual pode ser pior do que falhar explicitamente.

## Idempotência e duplicidade

Em sistemas com retries, filas e timeouts, a mesma operação pode chegar mais de uma vez. Uma operação idempotente produz o mesmo efeito quando repetida.

Uma forma comum de implementar isso é receber uma `idempotency_key`, armazenar o resultado associado a essa chave e devolver o mesmo resultado em novas tentativas. O registro da chave precisa ter uma política de expiração e ser gravado de forma consistente com a operação.

Idempotência é especialmente importante em:

- criação de pedidos;
- pagamentos;
- envio de notificações;
- processamento de eventos;
- jobs que podem ser retomados após uma falha.

## Redundância e recuperação

Uma instância única é um ponto único de falha. Para aumentar a disponibilidade, podemos distribuir instâncias em zonas de disponibilidade, regiões ou domínios de falha diferentes.

Mas redundância sozinha não resolve tudo. Também é preciso verificar saúde, remover instâncias defeituosas do tráfego, replicar dados e testar a recuperação.

- **Failover automático:** reduz o tempo de recuperação, mas pode promover uma réplica inadequada ou propagar uma falha.
- **Failover manual:** oferece mais controle, mas aumenta o RTO.
- **Replicação síncrona:** reduz perda de dados, mas pode aumentar latência e depender da disponibilidade de mais nós.
- **Replicação assíncrona:** mantém melhor desempenho, mas pode perder as últimas alterações durante uma falha.

Os conceitos de replicação e recuperação devem ser conectados aos requisitos de RPO e RTO:

- **RPO:** quanto de dados pode ser perdido.
- **RTO:** quanto tempo o serviço pode levar para voltar.

## Exemplo: checkout de um e-commerce

Imagine o fluxo:

```text
Cliente → API → Pedido → Estoque
                     ├── Pagamento
                     └── Notificação
```

Uma decisão possível é:

1. criar o pedido com uma chave idempotente;
2. reservar o estoque com timeout;
3. chamar o pagamento com limite de tentativas;
4. usar circuit breaker para proteger o serviço de pagamento;
5. publicar eventos para notificação de forma assíncrona;
6. enviar mensagens que falharem repetidamente para uma DLQ;
7. expirar reservas que não forem confirmadas.

O pagamento e a reserva de estoque podem exigir uma coordenação mais cuidadosa. Não é suficiente colocar retry em tudo: é necessário definir compensações quando uma etapa é concluída e outra falha. O padrão Saga é uma alternativa para esse tipo de fluxo e está descrito em [Saga](../10-Padroes-de-Arch/saga.md).

## Trade-offs

- **Disponibilidade x consistência:** continuar respondendo com cache ou dados antigos pode manter o serviço disponível, mas expõe informação desatualizada.
- **Retries x sobrecarga:** retries aumentam a chance de sucesso, mas podem piorar uma falha.
- **Timeout curto x operações incompletas:** liberar recursos rápido pode gerar cancelamentos e reprocessamento.
- **Redundância x custo:** mais réplicas e regiões aumentam disponibilidade, mas também custo e complexidade.
- **Isolamento x utilização:** separar pools reduz o raio de impacto, mas pode deixar capacidade ociosa.
- **Fallback x precisão:** uma resposta degradada melhora a experiência, mas pode ser incorreta para certos domínios.
- **Automação x controle:** failover automático reduz o tempo de resposta, mas pode tomar uma decisão ruim durante uma falha complexa.

O desenho resiliente é aquele que define conscientemente o comportamento durante a falha mais provável e durante a falha mais perigosa.

## Como aplicar em uma entrevista de System Design

Ao apresentar a arquitetura, pergunte:

1. Quais são os pontos únicos de falha?
2. O que acontece se cada dependência ficar lenta, indisponível ou retornar erro?
3. Qual operação pode ser repetida com segurança?
4. Onde aplicar timeout, retry, circuit breaker e limite de concorrência?
5. O fluxo pode ser assíncrono?
6. O sistema aceita consistência eventual ou precisa bloquear?
7. Qual é o RPO e o RTO?
8. Como detectar e recuperar uma mensagem presa ou uma réplica atrasada?

## Resumo

Resiliência é projetar o sistema para falhar de maneira previsível e limitar o impacto das falhas. Timeouts, retries com backoff, circuit breakers, bulkheads, rate limiting, filas, idempotência, fallback e redundância são ferramentas diferentes para problemas diferentes.

O trade-off central é que cada mecanismo de proteção adiciona custo, latência ou complexidade. A escolha deve partir dos requisitos do sistema, do impacto da falha e do nível de consistência e disponibilidade esperado.
