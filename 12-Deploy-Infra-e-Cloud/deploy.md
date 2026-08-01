# Tipos de Deploy

Pra realizar Deploy, temos duas principais abordagens:

- ***Deploy Manual*** x ***Deploy Automático***

No Deploy manual, normalmente a gente conecta na ec2 via ssh, da um pull do código e executa os comandos necessários para deploy, isso tem que ser feito manualmente e é ruim porque a pessoa pode errar e não executar todos os comandos necessários como verificar testes antes do deploy.
Já o Deploy automático, utiliza ferramentas como Jenkins, GitLab CI/CD ou GitHub Actions para automatizar o processo de deploy, garantindo que sejam executados várias pipelines (como testes, build e deploy) antes de realizar o deploy e tem uma precisão muito maior porque é sempre feito as mesmas verificações.  

## Formas de Deploy

### Recreate

Você recria a instância toda, desligando a atual e criando uma nova. É ruim porque tem downtime porque a instância é desligada e recriada.

### Rolling Update

Você atualiza a instância de forma gradual, desligando uma versão antiga e substituindo-a por uma nova. É bom porque tem downtime mínimo porque a instância é atualizada gradualmente.

### Blue/Green

Você tem duas instâncias, uma verde (a nova) e uma azul (a antiga), e você atualiza a verde antes de desligar a azul. É bom porque tem downtime mínimo porque a instância é atualizada gradualmente. A parte positiva é que não tem downtime e o RollBack é muito fácil porque como o tráfego é redirecionado para a nova instância, é fácil reverter para a antiga e as releases são mais seguras porque é possível testar a nova instância antes de desligar a antiga. Pontos negativos: Mais caro por ter 2 instâncias em execução ao mesmo tempo e um pouco mais complexo de configurar.

### Canary

Você tem uma instância em execução e atualiza-a gradualmente, testando-a antes de desligar a antiga. Funciona assim: inicialmente é direcionando 5% do tráfego para a nova instância e 95% para a antiga, ou seja, 5% do tráfego são pra BETA testers e 95% são pra a versão antiga. Após esse primeiro teste, a ideia é ir aumentando essa porcentagem de tráfego para a nova instância e diminuindo para a antiga, até que a nova instância atinja 100% do tráfego. Após isso, a nova instância é desligada e a antiga é ativada. Isso é uma estratégia muito boa porque permite verificar bugs em produção afetando apenas uma parte pequena da base de usuários e fácil de reverter caso necessário. Pontos negativos: Aumenta a complexidade.

#### Shadow

O Shadow é uma estratégia de canary que permite testar a nova instância sem afetar o tráfego real. Na prática funciona assim: É duplicado todas as requests de produção para a nova instância da nova versão, isso permite testar a nova instância sem afetar o tráfego real e conseguir fazer testes de carga, performance e estabilidade pra ver se essa nova versão realmente funciona como esperado e irá suportar a carga de produção. Lembrando que os requests não são verdadeiros, é uma simulação de requests de produção. Pontos negativos: Mais custoso que todas as outras estratégias porque duplica todas as requests de produção, então teria duas infras com a mesma carga.

### Serverless

Em serverless não nos preocupamos muito com o deploy da máquina física, pois o serviço é gerenciado pelo provedor e a infraestrutura é abstraída, o código novo é substituído automaticamente e não tem downtime.

---

## Dimensionamento e Infraestrutura (I/O Bound vs. CPU Bound)

No momento de planejar o deploy e provisionar a infraestrutura (tamanho de máquinas, auto-scaling e arquitetura), é fundamental entender se a aplicação é **I/O Bound** ou **CPU Bound**. Isso evita gastos desnecessários e quedas em produção.

### 1. I/O Bound (Limitado por Entrada e Saída)
A aplicação passa a maior parte do tempo esperando respostas de recursos externos (banco de dados, requisições de rede/APIs, leitura/escrita em arquivos, filas). A CPU fica ociosa na maior parte do tempo.

- **Exemplos:** Requisições CRUD padrão (ex: `GET /orders/123`), buscas no banco, consumo de APIs REST.
- **Estratégia de Infra/Deploy:**
  - Não exige CPUs super potentes. O foco é rede rápida e boa memória RAM.
  - Uso de Programação Assíncrona e Thread Pools para processar mais conexões simultâneas.
  - Implementação de Cache (ex: Redis) e Read Replicas no banco para diminuir a espera de I/O.
  - Escala Horizontal (adicionar mais réplicas leves da aplicação conforme a demanda de requisições aumenta).

### 2. CPU Bound (Limitado por Processamento)
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
