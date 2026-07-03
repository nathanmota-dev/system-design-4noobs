# SSE - Server Sent Events

Quando a gente quer uma atualização mais precisa, alterou no servidor, automaticamente já mude no nosso cliente, a gente pode usar a estratégia de SSE, onde a gente vai fazer uma requisião HTTP pro servidor e vai ficar escutando a resposta dele, então quando ele envia pra gente que alguma coisa mudou é atualizado em tempo real pro cliente. 

Exemplo: 

```javascript
const stream = new EventSource('/api/notifications');

stream.onmessage = (event) => {
    const notification = JSON.parse(event.data);
    showNotification(notification);
};

```

## Onde é usado?

- Mercado financeiro - Ver em tempo real o preço da ação, se aumentou se diminiu
- Gráficos - Sistemas que precisam de gráficos atualizando em tempo real


## Trade-off

- Só tráfega texto, não tráfega binário como Web sockets, então se sua aplicação precisa de imagens não é a melhor alternativa.
- Cliente não se comunica com servidor - Sempre do servidor para o cliente, a gente escuta o que o servidor nos envia, então para aplicações como chats não da pra usar essa estrátegia.