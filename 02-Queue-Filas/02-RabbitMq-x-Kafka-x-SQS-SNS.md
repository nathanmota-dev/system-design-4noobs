# RabbitMq x Kafka x SQS-SNS

Para conseguir definir qual é a melhor escolha para o projeto usado, precisamos entender algumas coisas, vamos começar entendendo como cada uma funciona.

---

## RabbitMq

O SQS e o SNS trabalham de uma forma onde o server empurra essa mensagem para o consumidor, ou seja, o trabalho mais árduo é feito pelo server, seguindo o seguinte fluxo

publisher -> publish -> exchange -> routes -> queue -> consumes -> consumer

---

## SQS e o SNS

O SQS e o SNS seguem o mesmo padrão do RabbitMq, onde a inteligência está no servidor, é isso que acontece por debaixo dos panos. A vantagem dele é conseguir utilizar outros serviços da AWS com uma facilidade de integração absurda por estarem no mesmo ambiente AWS, como por exemplo

SNS -> SQS -> Lambda -> RDS Aurora

---

## Kafka

Já o Kafka é totalmente diferente do padrão seguido pelo RabbitMq e pelo SQS, onde a inteligência fica no consumidor e não no servidor. O Broker ou server basicamente é um log, ele recebe a mensagem, armazena o log e cabe ao consumidor ler a mensagem na posição específica. O Consumer Group que vai saber o próximo offset (posição) que ele tem que ler naquela partição

---

Ou seja, RabbitMq e SQS/SNS seguem uma estrutura como uma pilha queue(FIFO), e nessa estrutura normalmente a gente trabalha com add() e pop() onde o add adiciona no começo da fila e o pop remove do fim, enquanto que Kafka segue uma estrutura como uma fila, onde temos um add para adicionar e para consumir damos um get(offset), ou seja, um get na posição. No Kafka a mensagem é enviada, o server recebe a mensagem, grava e devolve a partir do consumo que acontece em determinada posição onde cabe ao consumer ter o conhecimento. Já o Kafka segue um padrão diferente onde ele envia o log e o worker bate no log referenciando como um ponteiro e consome. A vantagem disso é que com Kafka você tem um log de toda sua fila e caso precise reenviar uma parte da fila é possível por esse log onde mesmo que o worker consuma o log é persistido

A principal diferença delas é no Throughput, Latência, Ordem, 1st Storage e Sec, podemos ver isso na imagem abaixo, porém vou detalhar cada uma nos próximos parágrafos.

![Comparacao](../assets/comparacao.png)

O throughput do RabbitMq é 10-50k por segundo (já é FIFO por padrão) com uma latência muito baixa por utilizar memória RAM apagando a mensagem apenas quando ela foi consumida, não tendo persistência.

O SQS com FIFO suporta apenas 3k por segundo, porém sem FIFO tem um suporte ilimitado e vale ficar ligado nos custos que podem aumentar significativamente, assim como qualquer serviço da AWS em escala e com uma latência bem maior comparado com seus concorrentes justamente pelo network traffic que ocorre na AWS e as mensagens são duráveis (como é durável não dá para saber, a AWS não especifica isso na doc). 

Já o Kafka pode suportar milhões de transações por segundo tendo essa vantagem caso o objetivo da sua aplicação seja milhões de transações por segundo, tem uma latência um pouco maior que o RabbitMq por fazer persistência no disco.

---

## Vantagens e Desvantagens

Começando com o RabbitMq, a principal desvantagem dele é que se forem publicadas muitas mensagens e não forem consumidas e ele utiliza memória RAM, com a memória cheia ele começa a utilizar o disco, o que diminui muito a performance dele. Ou seja, a principal vantagem do RabbitMq é que é muito útil em cenários que a gente vai publicar e consumir a mensagem em tempo real e não precisa de log de mensagem, além de ser bem simples de implementar comparado aos outros.

Problemas da AWS são com latência que é mais alta comparada às outras, custo que pode aumentar significativamente e caso precise de FIFO porque suporta apenas 3k por segundo. A maior vantagem é que ela tem uma coesão muito grande com outros serviços da AWS, então se a gente está utilizando outros serviços pode valer muito a pena.

Já o Kafka a principal desvantagem é que ele é pesado de manter por ter complexidade de cluster, ajuste de performance, escalabilidade manual, além de que comparado com todas as outras ferramentas é algo bem mais difícil, onde em cenários reais você vai ter um custo de mão de obra mais caro por precisar de SRE ou DevOps especializado por não ser fácil de implementar e com curva de aprendizado mais acentuada. Porém ele é muito vantajoso quando é necessário milhões de mensagens por segundo, persistência de mensagens e a capacidade de dar replay na fila, essa capacidade até dá com SQS também mas com um custo muito elevado, enquanto que com Kafka é bem mais tranquilo. 

## Segurança

Tanto RabbitMq, quanto Kafka utilizam TLS encryption e TLS/SASL auth enquanto que o SQS utiliza AWS KMS auth e AWS IAM.

---

## Qual é melhor?

Depois de analisar todos esses dados você pode estar se perguntando, qual é o melhor? E a resposta é: DEPENDE.

Assim como qualquer coisa na engenharia de software precisamos saber que tudo vai depender do contexto, onde se eu fosse analisar começaria por: 

- qual throughput eu preciso? 
- qual a latência que é necessária? 
- preciso de order? 
- preciso armazenar minha mensagem? 
- qual sec preciso? 
- preciso filtrar minhas mensagens? 
- preciso reenviar minhas mensagens?
- estou usando aws?

E com base nessas respostas conseguimos ter a visão de qual ferramenta de fila seria melhor para o contexto do nosso app.
