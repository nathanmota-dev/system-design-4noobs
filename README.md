<!-- Logo 4noobs -->

<p align="center">
  <a href="https://github.com/he4rt/4noobs" target="_blank">
    <img src="./assets/header_4noobs.svg">
  </a>
</p>

<!-- Title -->

<p align="center">
  <h1 align="center" style="font-size: 2.5em;">System Design Study</h1>

  <p align="center">
    <br />
    <a href="#roadmap"><strong>Explore a documentação »</strong></a>
    <br />
    <br />
    <a href="https://github.com/nathanmota-dev/system-design-study/issues">Report Bug</a>
    ·
    <a href="https://github.com/nathanmota-dev/system-design-study/issues">Request Feature</a>
  </p>
</p>

 <!-- ABOUT THE PROJECT -->

## **Aprenda os fundamentos de System Design e como projetar sistemas escaláveis, resilientes e distribuídos.**

Este repositório reúne estudos de System Design organizados em módulos, indo dos fundamentos de bancos de dados, redes e mensageria até arquiteturas distribuídas, observabilidade, segurança e cloud.

Ao percorrer o material, você encontrará conceitos, exemplos práticos e trade-offs para entender como os componentes de um sistema se relacionam e como tomar decisões arquiteturais conscientes.

O conteúdo foi organizado para servir como base de estudos, consulta e preparação para entrevistas de System Design.

<!-- ROADMAP OF PROJECT -->

## ROADMAP

| # | Módulo | Descrição |
|---|--------|-----------|
| 1 | [Início](./00-Inicio/) | Introdução e organização dos estudos |
| 1.1 | ┗ [init.md](./00-Inicio/init.md) | Ponto de partida do material |
| 2 | [Banco de Dados](./01-BD/) | Fundamentos, modelagem e otimização de bancos de dados |
| 2.1 | ┣ [01-BD.md](./01-BD/01-BD.md) | Fundamentos de bancos de dados |
| 2.2 | ┣ [02-SQLxNoSql.md](./01-BD/02-SQLxNoSql.md) | Comparação entre SQL e NoSQL |
| 2.3 | ┣ [03-Sharding-x-Partition.md](./01-BD/03-Sharding-x-Partition.md) | Diferenças entre sharding e partitioning |
| 2.4 | ┣ [04-0-Otimizacoes-de-DBs.md](./01-BD/04-0-Otimizacoes-de-DBs.md) | Visão geral de otimizações de bancos de dados |
| 2.5 | ┣ [04-01-Cache.md](./01-BD/04-01-Cache.md) | Uso de cache para melhorar o acesso aos dados |
| 2.6 | ┣ [04-02-Indices.md](./01-BD/04-02-Indices.md) | Índices e otimização de buscas |
| 2.7 | ┣ [04-03-Connection-Pooling.md](./01-BD/04-03-Connection-Pooling.md) | Reutilização e gerenciamento de conexões |
| 2.8 | ┣ [04-04-Replicacao-de-Leitura.md](./01-BD/04-04-Replicacao-de-Leitura.md) | Réplicas para distribuir operações de leitura |
| 2.9 | ┣ [04-05-Sharding-e-Partitioning.md](./01-BD/04-05-Sharding-e-Partitioning.md) | Estratégias de distribuição e particionamento de dados |
| 2.10 | ┣ [04-06-Teorema-CAP.md](./01-BD/04-06-Teorema-CAP.md) | Consistência, disponibilidade e tolerância a partições |
| 2.11 | ┣ [04-07-Normalizacao-e-Denormalizacao.md](./01-BD/04-07-Normalizacao-e-Denormalizacao.md) | Normalização e denormalização de dados |
| 2.12 | ┣ [04-08-Otimizacao-de-Queries.md](./01-BD/04-08-Otimizacao-de-Queries.md) | Técnicas para otimizar consultas |
| 2.13 | ┣ [04-09-Resumo.md](./01-BD/04-09-Resumo.md) | Resumo das otimizações de bancos |
| 2.14 | ┣ [05-Locks.md](./01-BD/05-Locks.md) | Locks e concorrência em bancos de dados |
| 2.15 | ┣ [06-NoSQL.md](./01-BD/06-NoSQL.md) | Conceitos e características de bancos NoSQL |
| 2.16 | ┗ [07-Federation.md](./01-BD/07-Federation.md) | Federação e distribuição entre bancos |
| 3 | [Filas](./02-Queue-Filas/) | Filas, mensageria e processamento assíncrono |
| 3.1 | ┣ [01-Message-Queue.md](./02-Queue-Filas/01-Message-Queue.md) | Conceitos de message queues |
| 3.2 | ┗ [02-RabbitMq-x-Kafka-x-SQS-SNS.md](./02-Queue-Filas/02-RabbitMq-x-Kafka-x-SQS-SNS.md) | Comparação entre RabbitMQ, Kafka, SQS e SNS |
| 4 | [Load Balancer](./03-Load-Balancer/) | Distribuição de tráfego entre servidores |
| 4.1 | ┗ [01-Load-Balancer.md](./03-Load-Balancer/01-Load-Balancer.md) | Funcionamento e estratégias de load balancing |
| 5 | [API Gateway](./04-API-Gateway/) | Entrada centralizada e gerenciamento de APIs |
| 5.1 | ┗ [01-API-Gateway.md](./04-API-Gateway/01-API-Gateway.md) | Responsabilidades e padrões de API Gateway |
| 6 | [Redes](./05-Network-Redes/) | Conceitos de networking e comunicação entre componentes |
| 6.1 | ┗ [01-Networking.md](./05-Network-Redes/01-Networking.md) | Fundamentos de redes |
| 7 | [Autenticação e Autorização](./06-Auth/) | Identidade, autenticação e controle de acesso |
| 7.1 | ┗ [index.md](./06-Auth/index.md) | Índice de autenticação e autorização |
| 8 | [DNS](./07-DNS/) | Resolução de nomes e direcionamento de requisições |
| 8.1 | ┗ [index.md](./07-DNS/index.md) | Índice de conceitos de DNS |
| 9 | [Sequenciadores](./08-Sequencer/) | Geração de identificadores e ordenação de eventos |
| 9.1 | ┗ [index.md](./08-Sequencer/index.md) | Índice de sequenciadores |
| 10 | [Blob Store, Cache e CDN](./09-Blob-Store-Cache-e-CDN/) | Armazenamento de objetos, cache e distribuição de conteúdo |
| 10.1 | ┣ [blob-store.md](./09-Blob-Store-Cache-e-CDN/blob-store.md) | Armazenamento de blobs e objetos |
| 10.2 | ┣ [cache.md](./09-Blob-Store-Cache-e-CDN/cache.md) | Conceitos e estratégias de cache |
| 10.3 | ┗ [cdn.md](./09-Blob-Store-Cache-e-CDN/cdn.md) | Content Delivery Networks |
| 11 | [Padrões de Arquitetura](./10-Padroes-de-Arch/) | Padrões e estratégias para estruturar sistemas |
| 11.1 | ┣ [CQRS.md](./10-Padroes-de-Arch/CQRS.md) | Separação entre leitura e escrita |
| 11.2 | ┣ [arquitetura-multi-tier.md](./10-Padroes-de-Arch/arquitetura-multi-tier.md) | Arquitetura em múltiplas camadas |
| 11.3 | ┣ [bff.md](./10-Padroes-de-Arch/bff.md) | Backend for Frontend |
| 11.4 | ┣ [design-alto-nivel.md](./10-Padroes-de-Arch/design-alto-nivel.md) | Design de sistemas em alto nível |
| 11.5 | ┣ [design-anti-patterns.md](./10-Padroes-de-Arch/design-anti-patterns.md) | Antipadrões de design de sistemas |
| 11.6 | ┣ [elastic-search.md](./10-Padroes-de-Arch/elastic-search.md) | Busca e indexação com Elasticsearch |
| 11.7 | ┣ [event-driven.md](./10-Padroes-de-Arch/event-driven.md) | Arquitetura orientada a eventos |
| 11.8 | ┣ [event-sourcing.md](./10-Padroes-de-Arch/event-sourcing.md) | Persistência de eventos e reconstrução de estado |
| 11.9 | ┣ [saga.md](./10-Padroes-de-Arch/saga.md) | Coordenação de transações distribuídas |
| 11.10 | ┗ [service-mesh.md](./10-Padroes-de-Arch/service-mesh.md) | Comunicação e observabilidade entre serviços |
| 12 | [Microserviços x Monolito](./11-Microservicos-x-Monolito/) | Comparação entre arquiteturas monolíticas e distribuídas |
| 12.1 | ┗ [microservicos-monolito.md](./11-Microservicos-x-Monolito/microservicos-monolito.md) | Características, vantagens e trade-offs |
| 13 | [Deploy, Infra e Cloud](./12-Deploy-Infra-e-Cloud/) | Implantação, infraestrutura e serviços de cloud |
| 13.1 | ┗ [deploy.md](./12-Deploy-Infra-e-Cloud/deploy.md) | Estratégias e práticas de deploy |
| 14 | [Observabilidade](./13-Observabilidade/) | Monitoramento, logs, métricas e rastreamento de sistemas |
| 14.1 | ┗ [index.md](./13-Observabilidade/index.md) | Logs, métricas, traces, alertas e SLOs |
| 15 | [Resiliência](./14-Resiliencia/) | Tolerância a falhas e recuperação de sistemas |
| 15.1 | ┗ [index.md](./14-Resiliencia/index.md) | Timeouts, retries, circuit breakers e degradação controlada |
| 16 | [Sistemas Distribuídos](./15-Sistemas-Distribuidos/) | Consistência, disponibilidade e coordenação distribuída |
| 16.1 | ┗ [index.md](./15-Sistemas-Distribuidos/index.md) | Replicação, particionamento, consenso e falhas parciais |
| 17 | [Segurança](./16-Seguranca/) | Práticas e mecanismos de segurança para sistemas e aplicações |
| 17.1 | ┗ [index.md](./16-Seguranca/index.md) | Threat modeling, identidade, autorização e proteção de dados |
| 18 | [Arquiteturas de Referência](./17-Arquiteturas-de-Referencia/) | Exemplos de arquiteturas de referência |
| 18.1 | ┗ [AWS.md](./17-Arquiteturas-de-Referencia/AWS.md) | Arquiteturas e serviços de referência na AWS |
| 19 | [Real-time](./18-Real-time/) | Comunicação e atualização de dados em tempo real |
| 19.1 | ┣ [1-Polling.md](./18-Real-time/1-Polling.md) | Atualização periódica por polling |
| 19.2 | ┣ [2-SSE.md](./18-Real-time/2-SSE.md) | Server-Sent Events |
| 19.3 | ┗ [3-Websockets.md](./18-Real-time/3-Websockets.md) | Comunicação bidirecional com WebSockets |
| 20 | [Requisitos e Estimativas](./19-Requistios-e-Estimativas/) | Requisitos, capacidade e planejamento de sistemas |
| 20.1 | ┣ [1-Requisitos.md](./19-Requistios-e-Estimativas/1-Requisitos.md) | Levantamento e definição de requisitos |
| 20.2 | ┣ [2-Redundancia.md](./19-Requistios-e-Estimativas/2-Redundancia.md) | Redundância e disponibilidade |
| 20.3 | ┗ [3-Estimativas.md](./19-Requistios-e-Estimativas/3-Estimativas.md) | Estimativas de capacidade e dimensionamento |
| 21 | [Como Montar um System Design](./20-Como-Montar-System-Design/) | Processo estruturado para construir e comunicar um System Design |
| 21.1 | ┗ [index.md](./20-Como-Montar-System-Design/index.md) | Roteiro de requisitos, arquitetura, escala, resiliência e trade-offs |
| 22 | [Exemplos de System Design](./21-Exemplos-de-System-Design/) | Exemplos práticos de sistemas e arquiteturas |
| 22.1 | ┣ [Encurtador.md](./21-Exemplos-de-System-Design/Encurtador.md) | Design de um encurtador de URLs |
| 22.2 | ┗ [WhatsApp.md](./21-Exemplos-de-System-Design/WhatsApp.md) | Design de um sistema de mensagens em tempo real |
| 23 | [Dicas](./22-Dicas/) | Dicas para entrevistas de System Design |
| 23.1 | ┗ [Dicas.md](./22-Dicas/Dicas.md) | Boas práticas para estruturar e apresentar uma solução |

<!-- CONTRIBUTING -->

## Como Contribuir

Contribuições fazem com que a comunidade open source seja um lugar incrível para aprender, inspirar e criar. Todas as contribuições são **extremamente apreciadas**.

1. Realize um fork do projeto.
2. Crie uma branch com a nova feature (`git checkout -b feature/featureBraba`).
3. Realize o commit (`git commit -m 'Adicionado conteudo brabo'`).
4. Realize o push da branch (`git push origin feature/featureBraba`).
5. Abra um Pull Request.

## Autor

- **Nathan Mota** - _Desenvolvedor Full Stack_ - [GitHub](https://github.com/nathanmota-dev)

---

<p align="center">
  <a href="https://github.com/he4rt/4noobs" target="_blank">
    <img src="./assets/footer_4noobs.svg" width="380">
  </a>
</p>
