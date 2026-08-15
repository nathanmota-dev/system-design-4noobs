# Tipos de Deploy

Para realizar deploy, temos duas abordagens principais:

- **Deploy manual:** uma pessoa executa as etapas necessárias.
- **Deploy automático:** uma pipeline executa etapas padronizadas.

No deploy manual, normalmente uma pessoa acessa o servidor, atualiza o código e executa os comandos necessários. O processo é mais sujeito a erros e etapas esquecidas, como testes ou migrações.

O deploy automático usa ferramentas como Jenkins, GitLab CI/CD ou GitHub Actions para executar uma pipeline reproduzível, com etapas como testes, build e implantação.

## Formas de Deploy

### Recreate

A versão atual é encerrada antes de a nova entrar em operação. É simples e barato, mas causa indisponibilidade durante a troca.

### Rolling Update

As instâncias antigas são substituídas gradualmente por novas. A estratégia mantém capacidade durante a atualização, mas as duas versões podem atender tráfego ao mesmo tempo e precisam ser compatíveis.

### Blue/Green

Dois ambientes equivalentes ficam disponíveis: um atende produção e o outro recebe a nova versão. Depois dos testes, o tráfego é redirecionado para o novo ambiente. O rollback é rápido enquanto o ambiente anterior continua disponível. O custo e a complexidade aumentam porque as duas infraestruturas coexistem durante a troca.

### Canary

Uma pequena parcela do tráfego, como 5%, é direcionada para a nova versão, enquanto o restante continua na versão anterior. Se métricas e testes estiverem saudáveis, a participação da nova versão aumenta até chegar a 100%. Em caso de problema, o tráfego volta para a versão anterior. A estratégia limita o impacto inicial de bugs, mas exige roteamento gradual, observabilidade e critérios de promoção e rollback.

### Shadow

No shadow deployment, uma cópia do tráfego real é enviada para a nova versão, mas suas respostas não chegam aos usuários. Isso ajuda a avaliar carga, desempenho e compatibilidade. A versão shadow não deve produzir efeitos colaterais reais, como cobranças ou envio de e-mails; o tráfego precisa ser anonimizado ou isolado quando contém dados sensíveis. O custo aumenta porque as duas versões processam carga simultaneamente.

### Serverless

Em serverless, o provedor abstrai os servidores, mas a estratégia de implantação continua importante. Versionamento, aliases, canary e rollback ajudam a evitar indisponibilidade e regressões durante a troca do código.

---

## Dimensionamento e Infraestrutura (I/O Bound vs. CPU Bound)

No momento de planejar o deploy e provisionar a infraestrutura (tamanho de máquinas, auto-scaling e arquitetura), é fundamental entender se a aplicação é **I/O Bound** ou **CPU Bound**. Isso evita gastos desnecessários e quedas em produção.

### 1. I/O Bound (limitado por entrada e saída)

A aplicação passa a maior parte do tempo esperando respostas de recursos externos (banco de dados, requisições de rede/APIs, leitura/escrita em arquivos, filas). A CPU fica ociosa na maior parte do tempo.

- **Exemplos:** Requisições CRUD padrão (ex: `GET /orders/123`), buscas no banco, consumo de APIs REST.
- **Estratégia de Infra/Deploy:**
  - Não exige CPUs super potentes. O foco é rede rápida e boa memória RAM.
  - Uso de Programação Assíncrona e Thread Pools para processar mais conexões simultâneas.
  - Implementação de Cache (ex: Redis) e Read Replicas no banco para diminuir a espera de I/O.
  - Escala Horizontal (adicionar mais réplicas leves da aplicação conforme a demanda de requisições aumenta).

### 2. CPU Bound (limitado por processamento)

O gargalo da aplicação é o processador. A CPU opera perto de 100% para realizar cálculos complexos e transformar dados.

- **Exemplos:** Processamento e conversão de imagem/vídeo, criptografia, Inteligência Artificial, Machine Learning, compressão de arquivos.
- **Estratégia de Infra/Deploy:**
  - Exige servidores com mais núcleos de CPU ou Hardware Especializado (GPUs).
  - A escala horizontal é baseada no uso de CPU (> 80%) ou tamanho da fila de tarefas, e não no número de requisições HTTP.
  - Uso de Workers/Job Queues para rodar o processamento pesado em segundo plano em instâncias isoladas, sem travar a API principal.
  - Processamento em Batches em horários de menor tráfego.

**Boas Práticas de Deploy:** Evite misturar tarefas I/O Bound e CPU Bound na mesma instância/container. Separe a API Web (I/O Bound) dos Workers de processamento (CPU Bound) em deploys isolados para evitar que uma tarefa pesada derrube a API inteira.

---

## Outras características

- Feature Flags: Flags que permitem ativar ou desativar funcionalidades sem precisar fazer um deploy.
- Diferença entre Deploy e Release: Não necessariamente todo deploy é uma release. Um deploy pode ser uma atualização parcial, correção de bug, ou uma feature oculta por feature flag. A release é a liberação oficial do recurso para os usuários.
