# Message Queue

Uma fila serve para armazenar mensagens que serão processadas posteriormente. Uma Message Queue, ou fila de mensagens, é uma estrutura que permite comunicação assíncrona entre processos. Em sistemas reais, normalmente temos aplicações ou serviços se comunicando com a fila. Em geral, filas têm estas características:

- Desacoplamento: o produtor não precisa conhecer diretamente quem vai consumir a mensagem nem em qual máquina esse consumidor está rodando.
- Produtor e consumidor: o produtor publica a mensagem na fila e o consumidor a processa. Eles podem rodar em servidores diferentes, o que é o mais comum, mas também podem estar na mesma máquina.
- Ordenação: algumas filas suportam FIFO (`First In, First Out`), mas isso depende da necessidade da aplicação e da tecnologia escolhida.
- Buffer de carga: a fila funciona como um buffer entre quem produz e quem consome mensagens. Em horários de pico, ela absorve o excesso de trabalho; depois, os consumidores vão drenando esse volume aos poucos.
- Fan-out: em alguns cenários, uma mesma mensagem pode ser replicada para múltiplas filas ou múltiplos consumidores, permitindo que diferentes processos reajam ao mesmo evento.
- Isolamento de falhas: se um consumidor falhar, o produtor pode continuar publicando mensagens, e o restante do sistema não necessariamente para por causa disso.
- Disponibilidade: a fila ajuda a manter o sistema operando mesmo quando alguma parte do processamento está degradada ou temporariamente indisponível.
- Processamento assíncrono: o produtor envia a mensagem e segue o fluxo sem esperar o processamento terminar naquele instante.

---

## Trade-offs

Filas não existem para deixar uma requisição do usuário mais rápida. O principal objetivo é desacoplar etapas do sistema, absorver picos de carga e dar resiliência ao processamento. Em muitos casos, inclusive, o fluxo completo fica mais lento, porque o trabalho deixa de ser imediato e passa a ser executado depois.

Esse é o principal trade-off: filas não são a melhor escolha quando o caso exige consistência forte e imediata, resposta síncrona ou latência muito baixa. Outro custo é a complexidade adicional de operar produtores, consumidores, retries, observabilidade e tratamento de falhas.

---

## Delivery semantics / Semântica de Entrega

- At-most-once: a mensagem é entregue no máximo uma vez. Nesse modelo, ela pode ser perdida se houver falha no meio do processamento. É útil quando perder um evento ocasionalmente é aceitável.
Exemplo de uso: streaming de vídeo ou métricas não críticas.
- At-least-once: a mensagem é entregue pelo menos uma vez. Esse é o modelo mais comum, porque prioriza não perder mensagens, mesmo que em alguns casos elas sejam processadas mais de uma vez.
Normalmente essa estratégia é combinada com idempotência. Exemplo: em pagamentos, a mensagem pode ser reenviada após uma falha; sem idempotência, o sistema corre o risco de cobrar duas vezes.
- Exactly-once: a mensagem é processada exatamente uma vez. É um objetivo desejável, mas caro e difícil de garantir de ponta a ponta em sistemas distribuídos.
Na prática, isso exige coordenação entre broker, consumidor e sistema de destino. Por isso, em muitos cenários o que se implementa de fato é `at-least-once` com idempotência, que entrega um resultado mais realista e operacionalmente viável.

---

## Dead Letter Queue

Uma Dead Letter Queue (DLQ) é uma fila especial usada para armazenar mensagens que não puderam ser processadas corretamente. A ideia é simples: se uma mensagem falhar várias vezes, ou falhar de uma forma não recuperável, ela sai da fila principal e vai para a DLQ. Depois disso, o sistema ou a equipe pode analisar o problema e decidir o tratamento adequado.

---

## Queue Partitioning

Vamos supor um sistema gerador de PDFs. Esse sistema recebe eventos para processar arquivos, e esses eventos são consumidos por servidores ou instâncias de processamento. Em um primeiro momento, dá para escalar os consumidores aumentando a máquina ou adicionando mais instâncias. O problema aparece quando a própria fila vira gargalo.

Nesse ponto entra o particionamento de filas. Em vez de concentrar tudo em uma única fila, o sistema divide a carga entre várias partições ou filas, seguindo uma regra de roteamento. No exemplo da imagem abaixo, uma regra simples seria:

- IDs pares vão para uma fila
- IDs ímpares vão para outra

Essa é só uma forma de particionar. Dependendo do caso, a divisão pode ser por cliente, região, tipo de evento, prioridade ou qualquer outra chave que distribua melhor a carga.

![Queue-Partitioning](../assets/Queue-Partitioning.png)

 ---

 ## Consumers ou Consumidores

Muita gente pensa na fila como se ela "empurrasse" a mensagem para o consumidor. Em muitos sistemas, o comportamento real é o contrário: o consumidor faz um `pull` da mensagem. Isso acontece porque a fila nem sempre sabe quando o processamento terminou com sucesso; quem sabe disso é o consumidor.

O fluxo costuma ser este:

1. Buscar a mensagem na fila
2. Processar a mensagem
3. Avisar que o processamento terminou (`acknowledge`)
4. Remover a mensagem da fila ou marcar o offset como consumido

É nesse ponto que entram as semânticas de entrega, como `at-most-once` e `at-least-once`.

Sistemas como o Kafka usam offset. Nesse modelo, o consumidor registra qual foi a última mensagem processada e, com base nisso, consegue retomar a leitura do ponto correto.

---

### Escalabilidade de filas - Como escalar?

Para escalar um sistema baseado em filas, normalmente aumentamos a capacidade de processamento paralelo ou dividimos melhor a carga. As principais estratégias são:

1. Particionamento de filas
2. Mais filas
3. Mais consumidores
4. Processamento em batch: em vez de processar uma mensagem por vez, o sistema processa várias em conjunto
5. Autoscaling: aumentar ou reduzir consumidores de acordo com a carga
6. Filas com prioridade: separar eventos mais urgentes dos menos urgentes, conforme a regra de negócio

---

## Problemas com escalabilidade de filas

1. Pode acontecer de uma fila ficar sobrecarregada enquanto outra fica ociosa. Nesse caso, a estratégia de particionamento ou roteamento pode estar ruim.
2. O debugging fica mais difícil, porque a mensagem pode passar por várias filas, consumidores, tentativas e reprocessamentos.

---

## DLQ e Retries

Vamos supor que um consumidor precisa enviar um e-mail para o cliente, mas a API externa de e-mail falha. O que fazer?

Uma abordagem comum é usar retries automáticos. O sistema tenta processar a mensagem novamente após um intervalo de tempo, até que a dependência volte a funcionar ou o limite de tentativas seja atingido. A regra exata depende do negócio, da criticidade da operação e da política de retries definida.

Um ponto importante aqui é a idempotência. Se o sistema vai tentar de novo, ele precisa garantir que reprocessar a mesma mensagem não gere duplicidade de efeito.

![DQL-Retry](../assets/DQL-Retry.png)

Se a mensagem continuar falhando e o número máximo de tentativas for atingido, ela pode ser enviada para a Dead Letter Queue. A partir daí, o sistema pode:

- registrar o erro em monitoramento
- persistir a mensagem para análise posterior
- disparar alertas internos
- iniciar algum fluxo de tratamento manual ou automático

Na imagem, a ideia é exatamente essa: a mensagem sai da fila principal, passa pelos consumidores, pode sofrer reenvio (`resend`) e, se ainda assim falhar, vai para a DLQ.
