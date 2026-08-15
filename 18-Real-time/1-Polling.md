# Polling

Polling é uma técnica na qual, a cada intervalo, o cliente dispara uma requisição HTTP para verificar se houve alterações.

Exemplo:

```javascript
setInterval(async () => {
    const response = await fetch('/api/notifications/new');
    const newNotifications = await response.json();

    if (newNotifications.length > 0) {
        updateNotifications(newNotifications);
    }
}, 5000);
```

Nesse exemplo, a cada cinco segundos é enviada uma requisição para verificar se chegou uma notificação.

## Onde é usado?

O polling é usado em:

- Notificações que toleram algum atraso.
- Relatórios: enquanto o servidor processa um PDF, o cliente consulta periodicamente o status.
- Fluxos não críticos, que não precisam de atualização instantânea.

## Trade-off

- **Latência de atualização:** uma mudança pode levar até o próximo intervalo para aparecer no cliente.
- **Desperdício de recursos:** muitas requisições retornam sem novidades. Em grande escala, esse tráfego pode gerar carga significativa nos servidores.
- **Simplicidade:** usa HTTP convencional, funciona bem com proxies e é fácil de implementar e depurar.

## Cenários

Em entrevistas, polling costuma ser um bom ponto de partida quando o atraso é aceitável. Ele também pode funcionar em sistemas grandes, desde que o intervalo, cache, backoff e capacidade estejam bem dimensionados. Quando o requisito exige atualizações frequentes e imediatas, SSE ou WebSockets podem ser opções melhores.
