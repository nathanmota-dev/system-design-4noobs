# Web sockets

- Permite conexões Bi-direcionais
- Latência baixíssima
- Trafega textos e frames binários

Exemplo: 

```javascript
const chat = new WebSocket('wss://chat.myapp.com/ws');

chat.onmessage = (event) => {
    const msg = JSON.parse(event.data);
    appendMessage(msg); // Ana: "Opa, tudo bem?"
};

chat.send(JSON.stringify({
    type: 'message',
    text: 'Tudo certo!'
}));
```

O protocolo é diferente, a requisição é assim. A requisição é HTTP porém no cabeçalho da requisição ele manda isso.

```text
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: ...
```

Ele abre a requisão HTTP e depois ele muda a conexão para uma conexão TCP.

## Onde é usado?

- Chats
- Colaboração em tempo real (como google docs, figma)


## Trade-off

- Complexto, precisa de infra específica.

Exemplo:

Um servidor com

```text
- 16 cores
- 32GB RAM
- Mensagens com média de 1KB (1000 caracteres)
- 100k mensagens totais no servidor p/s (com base em todos os usuarios conectados)
```

Um servidor com essas configurações consegue ageuntar de 300k a 500k simultaneas, só que um Whatsapp precisa de muito mais.

Para a replicação tem um fator único, não basta apenas replicar as instâncias e colocar um Load Balancer, porque o LB padrão atua na camada 7, porém esse LB precisa ser específico e operar na camada 4.

![LB com Websockets](../assets/websockets.png)

Não pode ser na camada 7 porque o Load Balancer pega a requisição, verifica ela, lê os cabeçalhos, empacota de novo e faz outra chamada pro servidor redirecionando a requisição, já o layer 4 não faz isso e apenas abre a requisição redirecionando pro servidor com menos carga. 


## Problemas de System Design

Como resolver quando um usuario esta conectado em um servidor com essa conexão de websockets aberta a manda uma mensagem pra um usuário em outro servidor?f

PUB/SUB com Redis!

user 1 se conecta: SUBSCRIBE user:1
user 2 se conecta: SUBSCRIBE user:2

user 1 manda mensagem: PUBLISH user:2 "mensagem do 1"
user 2, por estar inscrito no tópico, recebe "mensagem do 1"
