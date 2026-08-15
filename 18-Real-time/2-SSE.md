# SSE — Server-Sent Events

Quando uma alteração do servidor precisa chegar ao cliente rapidamente, podemos usar SSE. O cliente abre uma conexão HTTP de longa duração e passa a receber eventos enviados pelo servidor. A comunicação é unidirecional: servidor para cliente.

Exemplo: 

```javascript
const stream = new EventSource('/api/notifications');

stream.onmessage = (event) => {
    const notification = JSON.parse(event.data);
    showNotification(notification);
};

```

## Onde é usado?

- Mercado financeiro, para acompanhar atualizações de preços.
- Gráficos e dashboards atualizados continuamente.
- Notificações e progresso de tarefas longas.


## Trade-off

- **Formato textual:** SSE usa `text/event-stream`. Dados binários precisam ser codificados ou entregues por outro endpoint.
- **Comunicação unidirecional:** o cliente ainda pode enviar dados por requisições HTTP separadas, mas a conexão SSE só transporta eventos do servidor para o cliente.
- **Reconexão simples:** o navegador pode reconectar automaticamente e enviar `Last-Event-ID`, mas a aplicação precisa definir retenção e retomada dos eventos.
