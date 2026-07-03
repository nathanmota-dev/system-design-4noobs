# Polling

Polling é uma técnica que eu já implementei, onde a cada x segundos a gente dispara uma requisição HTTP para verificar alterações.

Exemplo:

```javascript
setInterval(async () => {
    const newNotifications = await fetch('/api/notifications/new');
    if (newNotifications.length > 0) {
        updateNotifications(newNotifications);
    }
}, 5000);
```

Nesse exemplo a cada 5 segundos é enviado uma requisição para verificar se a notificação chegou.

## Onde é usado?

O polling é usado em:

- Notificações 
- Relatórios - PDF tem um tempo de processamento pelo servidor, enquanto o servidor não termina de processar o pdf é feito um pooling verificando isso.
- Fluxos não críticos - fluxos que não precisam saber exatamente no momento que uma notificação chegou como por exemplo uma mensagem do WhatsApp, ou quando a gente pede um Uber.

## Trade-off

- Alta latência, pouca precisão - Se a gente tem um sistema que precisa de dados atualizando em tempo real com precisão o pooling não é uma boa estratégia porque eu só teria a atualização do estado a cada x segundos.
- Desperdício de recursos - Várias requisições são feitas e são desperdícadas, ou seja, elas não servem pra nada porque aquilo esperado não foi retornado, se a gente está falando de um sistema pequeno isso não teria problemas, porém um sistema que tem 2 milhões de acesso naquele serviço e é usado polling, muita carga nos servidores poderia ser evitada.

## Cenários

Em entrevistas pooling tem que ser a primeira coisa a se pensar, porém só é válido para sistemas pequenos e que não precisam de atualização em tempo real 