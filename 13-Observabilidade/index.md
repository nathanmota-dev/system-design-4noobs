# Observabilidade

Observabilidade é a capacidade de entender o estado interno de um sistema a partir dos sinais que ele produz. Em um sistema pequeno, talvez seja possível descobrir um problema olhando a aplicação diretamente. Em um sistema distribuído, com vários serviços, bancos, filas e regiões, isso deixa de ser suficiente.

Em System Design, observabilidade não deve ser tratada como algo que será adicionado depois do deploy. Ela faz parte da arquitetura: precisamos decidir quais eventos serão registrados, quais métricas serão coletadas, como uma requisição será acompanhada entre serviços e quais sinais indicarão que o sistema está degradando.

## Por que observabilidade é importante?

Considere uma requisição que chega ao cliente com erro. O problema pode estar:

- no cliente ou na rede;
- no API Gateway;
- no serviço de aplicação;
- em uma chamada para outro serviço;
- no banco de dados;
- em uma fila com atraso;
- em uma dependência externa.

Sem contexto suficiente, a equipe apenas sabe que a requisição falhou. Com observabilidade, ela consegue responder perguntas como:

- Qual endpoint está falhando?
- O erro afeta todos os usuários ou apenas uma região?
- O problema começou depois de um deploy?
- A latência aumentou no serviço ou no banco?
- Existe uma fila acumulando mensagens?
- A falha é causada por uma dependência externa?

O objetivo não é coletar o máximo de dados possível. O objetivo é coletar os dados necessários para tomar decisões e investigar incidentes.

## Monitoramento e observabilidade

Os termos são relacionados, mas não são exatamente iguais.

- **Monitoramento:** acompanha sinais conhecidos e verifica se eles estão dentro de limites esperados.
- **Observabilidade:** permite investigar comportamentos inesperados usando os sinais produzidos pelo sistema.

Um alerta de que a taxa de erro passou de 5% é monitoramento. Conseguir seguir uma requisição específica e descobrir que ela passou por um serviço lento e por uma consulta sem índice é observabilidade.

Os dois são necessários. Monitoramento ajuda a detectar o incidente; observabilidade ajuda a entender a causa e o impacto.

## Os principais sinais

### Logs

Logs são registros de acontecimentos. Eles ajudam a explicar o que a aplicação estava fazendo em determinado momento.

Um log útil normalmente contém:

- data e hora em UTC;
- nível (`debug`, `info`, `warn` ou `error`);
- nome do serviço e versão;
- ambiente e região;
- `request_id` ou `trace_id`;
- tipo do evento;
- informações relevantes para investigação;
- erro e stack trace quando aplicável.

É preferível usar logs estruturados, normalmente em JSON, em vez de mensagens livres. Assim, é possível filtrar por serviço, endpoint, código de resposta ou identificador da requisição.

```json
{
  "level": "error",
  "service": "orders",
  "event": "payment_authorization_failed",
  "trace_id": "abc-123",
  "order_id": "order-456",
  "error_code": "payment_timeout"
}
```

Logs não devem conter senha, token, chave privada ou dados pessoais desnecessários. A observabilidade precisa respeitar os requisitos de segurança e privacidade do sistema.

### Métricas

Métricas são valores numéricos agregados ao longo do tempo. Elas são eficientes para acompanhar tendências e gerar alertas.

Algumas métricas importantes:

- **Throughput:** quantidade de requisições, eventos ou mensagens processadas.
- **Latência:** tempo de resposta, preferencialmente com percentis como P50, P95 e P99.
- **Taxa de erro:** proporção de respostas ou operações que falharam.
- **Saturação:** quanto um recurso está próximo do limite, como CPU, memória, conexões ou espaço em disco.
- **Fila:** tamanho, idade da mensagem mais antiga e taxa de processamento.

Uma métrica precisa ter dimensões úteis. Colocar `user_id` como dimensão de todas as métricas, por exemplo, pode gerar cardinalidade muito alta e aumentar bastante o custo de armazenamento e consulta.

### Traces

Um trace representa o caminho de uma requisição ou operação distribuída. Ele é formado por vários spans, e cada span representa uma etapa desse caminho.

Exemplo:

```text
Cliente
  └── API Gateway
        └── Orders Service
              ├── Inventory Service
              └── Banco de Dados
```

Se cada etapa propagar o mesmo `trace_id`, será possível visualizar onde o tempo foi gasto e qual dependência causou a falha.

Tracing é especialmente útil quando uma requisição passa por vários serviços. Em contrapartida, gerar e armazenar traces para 100% do tráfego pode ser caro. Uma alternativa é aplicar sampling, mantendo uma amostra maior para erros, requisições lentas e fluxos críticos.

## Exemplos de ferramentas

Não existe uma única ferramenta que resolva toda a observabilidade. Em geral, a arquitetura combina uma ferramenta de instrumentação, backends para cada sinal e uma interface para consulta e alertas.

| Necessidade | Exemplos | Uso típico |
|---|---|---|
| Instrumentação e coleta | **OpenTelemetry** | Instrumentar aplicações e infraestrutura e enviar logs, métricas e traces para diferentes backends por meio de SDKs e do Collector. |
| Métricas e alertas | **Prometheus** e **Alertmanager** | Armazenar séries temporais, consultar métricas e disparar ou encaminhar alertas baseados em regras. |
| Dashboards | **Grafana** | Criar dashboards, consultar múltiplas fontes e correlacionar métricas, logs e traces. |
| Logs | **Loki**, **Elasticsearch/OpenSearch**, **Kibana/OpenSearch Dashboards** | Centralizar, pesquisar e analisar logs; agentes como Fluent Bit ou Vector podem fazer a coleta e o encaminhamento. |
| Tracing distribuído | **Jaeger**, **Grafana Tempo** e **Zipkin** | Armazenar e visualizar traces e spans para encontrar gargalos e dependências lentas. |
| Erros e APM | **Sentry**, **Datadog**, **New Relic** e **Dynatrace** | Reunir erros, stack traces, métricas de aplicação, traces e informações de desempenho em uma solução integrada. |
| Operação de incidentes | **PagerDuty**, **Opsgenie** e **Grafana OnCall** | Encaminhar alertas, aplicar escalonamento e organizar a resposta da equipe de plantão. |
| Serviços gerenciados de nuvem | **Amazon CloudWatch**, **Azure Monitor** e **Google Cloud Observability** | Coletar sinais dos serviços da própria nuvem com menor esforço de operação, geralmente com integração aos demais componentes. |

Um exemplo de stack é usar OpenTelemetry para instrumentação, Prometheus para métricas, Loki para logs, Grafana Tempo para traces e Grafana para dashboards. Em uma plataforma gerenciada, parte desses componentes pode ser substituída por um produto de APM, mas continuam sendo importantes a propagação de contexto, a definição dos alertas e o controle de custo e retenção.

## Correlação entre sinais

Logs, métricas e traces são mais úteis quando podem ser relacionados.

Um fluxo comum é:

1. O gateway cria ou recebe um `trace_id`.
2. O identificador é propagado nos headers das chamadas internas.
3. Cada serviço inclui o identificador nos logs.
4. Métricas agregadas indicam que a latência ou a taxa de erro aumentou.
5. O trace e os logs ajudam a investigar uma requisição representativa.

Além do `trace_id`, pode existir um `request_id` para identificar uma requisição específica. Em sistemas assíncronos, o contexto também precisa acompanhar a mensagem publicada na fila.

## Health checks

Health checks informam se uma instância está apta a receber tráfego ou se ainda está viva.

- **Liveness:** verifica se o processo está vivo. Se falhar, pode ser necessário reiniciar a instância.
- **Readiness:** verifica se a instância está pronta para receber tráfego. Ela pode estar viva, mas sem conexão com o banco ou ainda carregando configuração.
- **Startup:** ajuda a diferenciar uma aplicação que ainda está inicializando de uma aplicação travada.

Um erro comum é fazer o liveness depender de todas as dependências externas. Se o banco cair, todas as instâncias podem parecer mortas e serem reiniciadas em loop. A decisão precisa considerar o comportamento desejado e o risco de uma tempestade de reinicializações.

## SLI, SLO e SLA

- **SLI (Service Level Indicator):** indicador medido, como taxa de sucesso ou P95 de latência.
- **SLO (Service Level Objective):** objetivo interno para esse indicador, como 99,9% de requisições bem-sucedidas.
- **SLA (Service Level Agreement):** compromisso formal com o cliente, normalmente com consequências caso não seja cumprido.

Um SLO ajuda a transformar a ideia de confiabilidade em uma meta mensurável. A partir dele, pode-se calcular o error budget: a parcela de falhas que ainda é aceitável no período.

Se o sistema tem um SLO de 99,9% de disponibilidade, ele possui uma margem de indisponibilidade. Essa margem pode ser usada para lançar mudanças com segurança, desde que o sistema não esteja consumindo o orçamento de erros rapidamente.

## Alertas

Um alerta deve indicar um problema que exige ação. Alertar para cada pequena variação gera fadiga e aumenta a chance de a equipe ignorar um incidente real.

Alertas úteis normalmente estão ligados a:

- erro percebido pelo usuário;
- aumento sustentado de latência;
- queda de throughput;
- saturação de recurso;
- crescimento anormal de fila;
- falha de réplica ou perda de capacidade;
- consumo acelerado do error budget.

É importante diferenciar alerta de diagnóstico. A equipe pode ser alertada de que a taxa de erro aumentou; a investigação pode mostrar que o banco está sem conexões disponíveis.

## Arquitetura de observabilidade

Um desenho comum é:

```text
Aplicações e infraestrutura
          │
          ▼
Agentes ou OpenTelemetry Collector
          │
          ▼
Pipeline de coleta e processamento
          │
          ├── Backend de logs
          ├── Backend de métricas
          └── Backend de traces
                    │
                    ▼
          Dashboards e alertas
```

O pipeline pode fazer enriquecimento, filtragem, amostragem e mascaramento de dados. Em alguns casos, um buffer ou uma fila desacopla a aplicação do backend de observabilidade, evitando que uma indisponibilidade desse backend bloqueie o tráfego principal.

## Trade-offs

Observabilidade melhora a capacidade de operar o sistema, mas não é gratuita.

- **Mais dados x mais custo:** aumentar retenção, resolução e volume de sinais eleva o custo de armazenamento e processamento.
- **Detalhe x privacidade:** logs muito detalhados podem expor dados sensíveis.
- **Tracing completo x sampling:** coletar tudo facilita a investigação, mas pode ser caro; sampling reduz custo, mas pode perder casos raros.
- **Centralização x dependência:** uma plataforma central facilita consultas, mas cria dependência operacional e pode concentrar custos.
- **Alertas sensíveis x fadiga:** limiares muito agressivos detectam mais variações, mas geram mais falsos positivos.
- **Instrumentação x desempenho:** instrumentar cada operação aumenta visibilidade, mas pode adicionar overhead.

O melhor desenho é o que dá visibilidade suficiente para os riscos e requisitos do sistema, sem transformar a coleta em um novo gargalo.

## Como aplicar em uma entrevista de System Design

Ao desenhar um sistema, vale perguntar:

1. Quais indicadores representam sucesso para o usuário?
2. Como medir latência, erro e throughput de cada componente?
3. Como seguir uma requisição entre serviços síncronos e assíncronos?
4. O que precisa gerar alerta imediato?
5. Qual dado não pode ser registrado por segurança ou privacidade?
6. Qual retenção e nível de detalhe são realmente necessários?
7. O que acontece se a plataforma de observabilidade estiver indisponível?

Uma boa resposta mostra que o sistema não deve apenas funcionar; ele também precisa ser operável e diagnosticável quando algo der errado.

## Resumo

Observabilidade combina logs, métricas e traces para ajudar a entender o comportamento de um sistema. Em arquiteturas distribuídas, ela depende de correlação, propagação de contexto, alertas baseados em objetivos e cuidado com custo e privacidade.

O trade-off central é equilibrar visibilidade, custo, desempenho e segurança. A observabilidade deve ser desenhada junto com o sistema, porque descobrir a causa de uma falha depois do incidente é muito mais difícil quando os sinais não foram coletados desde o início.
