# Load Balancer

O propósito do Load Balancer é distribuir o tráfego de entrada entre vários servidores ou instâncias de um serviço. Então, quando um usuário acessa uma aplicação, o Load Balancer decide para qual instância aquela requisição será enviada.

![LoadBalancer](../assets/LoadBalancer.png)

### Pra que serve o Load Balancer?

1. Distribuir carga - Distribuir o tráfego de entrada entre vários servidores ou instâncias de um serviço.
2. Escalabilidade - Permitir que a aplicação cresça horizontalmente, adicionando ou removendo instâncias conforme a demanda.
3. Alta disponibilidade - Se uma instância falhar, o tráfego pode ser redirecionado para outras que continuam saudáveis.
4. Segurança - Em alguns cenários, ele também ajuda em camadas de proteção, como TLS termination, WAF e mitigação de ataques volumétricos.

Em aplicações reais, é comum existir mais de um Load Balancer. Podemos ter um entre cliente e front-end, outro entre front-end e back-end e, em algumas arquiteturas, mecanismos de balanceamento também na camada de banco para distribuir leitura entre réplicas. Ou seja, o balanceamento pode aparecer em vários níveis da aplicação.

---

### Funcionalidades do Load Balancer:

1. Health check - Verificar a saúde das instâncias. Se uma instância cair ou começar a responder mal, o Load Balancer para de enviar tráfego para ela.
2. Distribuição de carga - Direcionar mais tráfego para instâncias menos ocupadas, dependendo do algoritmo configurado.
3. TLS termination - Encerrar o TLS no próprio balanceador, centralizando certificados e simplificando a camada de aplicação.
4. Regras de roteamento - Em balanceadores de camada 7, é possível rotear por host, caminho, header ou outras características da requisição.
5. Segurança - Integrar autenticação, rate limiting, WAF ou outras proteções, dependendo da solução adotada.

---

### Como o Load Balancer funciona distribuindo a carga?

Agora que entendemos as funcionalidades do Load Balancer, vamos entender como ele funciona distribuindo a carga:

1. Round Robin - O Load Balancer distribui a carga entre os servidores ou instâncias de um serviço usando o algoritmo Round Robin, onde cada solicitação é atribuída ao próximo servidor na fila.
2. Weighted Round Robin - O Load Balancer pode usar o algoritmo Weighted Round Robin, onde cada servidor recebe um peso e o tráfego é distribuído com base nesses pesos. Isso é útil quando as instâncias têm capacidades diferentes.
3. Least Connections - O Load Balancer pode usar o algoritmo Least Connections, onde o servidor ou instância de um serviço com menos conexões ativas recebe mais tráfego.
4. Least Response Time - O Load Balancer pode usar o algoritmo Least Response Time, onde o servidor ou instância de um serviço com menor tempo de resposta recebe mais tráfego.
5. IP Hash - O Load Balancer pode usar o algoritmo IP Hash, onde o servidor é escolhido com base no endereço IP do cliente. Isso ajuda a manter afinidade, direcionando o mesmo cliente para a mesma instância em muitos casos.

### Estático vs Dinâmico

O Load Balancer pode ser configurado como estático ou dinâmico, dependendo da necessidade. 

Um Load Balancer estático tem:
1. Regras fixas - O conjunto de destinos é previamente definido e muda pouco ou nada em tempo de execução.
2. Pouca ou nenhuma adaptação ao estado atual - Ele tende a decidir com base em regras fixas, sem reagir bem à saúde ou à carga das instâncias.

Um Load Balancer dinâmico tem:
1. Monitoramento dos servidores - Ele acompanha a saúde e, em alguns casos, a carga das instâncias.
2. Ajuste em tempo real - Direciona o tráfego para os destinos mais adequados com base no estado atual do sistema.


### Stateful vs Stateless

Um Load Balancer stateful mantém algum contexto da sessão do cliente, como afinidade de sessão (`sticky session`). Já um Load Balancer stateless trata cada requisição de forma independente, sem guardar esse contexto.

Na prática, arquiteturas stateless costumam escalar melhor, porque qualquer instância consegue atender qualquer requisição. Quando a aplicação depende de sessão local, o balanceador pode precisar manter afinidade entre cliente e servidor.

---

### Camada OSI

O Load Balancer normalmente opera na camada 4 ou na camada 7 do modelo OSI.

- Camada 4: toma decisões com base em IP, porta, TCP e UDP
- Camada 7: toma decisões com base em informações da aplicação, como URL, host, header e protocolo HTTP/HTTPS

A imagem destaca exatamente essa diferença: na camada 7 o balanceador enxerga detalhes do protocolo de aplicação; na camada 4 ele trabalha em um nível mais baixo, olhando principalmente para endereço e transporte.

![OSI-LB](../assets/OSI-LB.png)

