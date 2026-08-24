# Como montar um System Design

Em uma entrevista, o objetivo não é desenhar um sistema pronto para produção em uma hora. O objetivo é mostrar um raciocínio claro: entender o problema, explicitar as premissas, propor uma solução simples e aprofundar os pontos que oferecem maior risco.

Use o roteiro abaixo para manter o design objetivo e no mesmo formato dos exemplos deste módulo.

**1 - Requisitos Funcionais**

Liste o que o sistema precisa fazer, priorizando o fluxo principal:

* quem usa o sistema e quais operações cada ator executa;
* entradas, saídas e estados importantes;
* funcionalidades que entram no escopo;
* funcionalidades que ficam fora do escopo.

Não escolha tecnologias nesta etapa. Primeiro defina o problema.

**2 - Requisitos Não Funcionais**

Registre como o sistema precisa se comportar:

* quantidade de usuários, requisições e dados;
* latência esperada, de preferência com P95 ou P99;
* disponibilidade, durabilidade e recuperação;
* consistência, ordenação e tolerância a perda ou atraso;
* crescimento, regiões, segurança e custo.

Sempre que possível, transforme requisitos vagos em números ou prioridades.

**3 - Pontos de Atenção**

Liste os riscos que podem mudar a arquitetura:

* gargalos, picos de tráfego e pontos únicos de falha;
* consistência, concorrência, ordenação e duplicidade;
* retries, idempotência, timeouts e backpressure;
* processamento síncrono versus assíncrono;
* cache, invalidação, particionamento e retenção;
* segurança, abuso, privacidade e dependências externas.

Cada ponto de atenção deve resultar em uma decisão ou ser explicitamente deixado para um aprofundamento posterior.

**4 - Estimativas (BoE)**

Faça contas de ordem de grandeza para dimensionar a solução:

* RPS médio e de pico;
* proporção entre leituras e escritas;
* armazenamento inicial, crescimento e replicação;
* tráfego, conexões simultâneas e mensagens pendentes;
* capacidade necessária durante falhas e recuperação.

Fórmulas úteis:

```text
RPS médio = requisições no período / segundos do período
Armazenamento = volume de dados × tamanho médio × retenção × replicação
```

Diferencie média de pico, declare as premissas e identifique qual componente tende a ser o primeiro gargalo.

**5 - Overview de Alto Nível**

Comece com o caminho mínimo que atende aos requisitos:

```text
[Cliente] -> [Gateway / Load Balancer] -> [Serviço] -> [Banco]
                                                \-> [Cache / Fila, se necessário]
```

Depois, substitua os blocos pelos componentes do domínio. Mostre o sentido das leituras, escritas e eventos. Só adicione CDN, réplicas, filas, workers ou sharding quando um requisito ou uma estimativa justificar a complexidade.

**6 - Schema DB**

Modele os dados necessários para os fluxos principais:

* entidades, relacionamentos e fonte de verdade;
* chaves, índices e padrões de acesso;
* identificadores e estratégia de particionamento;
* dados transacionais, derivados e armazenados em cache;
* expiração, arquivamento, deleção e retenção.

O modelo deve suportar as consultas definidas nas APIs. Não escolha o banco apenas pela popularidade.

**7 - Design da API**

Para cada operação importante, mostre:

* método, recurso, parâmetros e corpo;
* resposta, códigos de erro e autenticação;
* paginação, filtros e limites;
* idempotência, retries e versionamento.

O contrato deve deixar claro quais operações são síncronas e quais geram processamento posterior.

**8 - Flow**

Desenhe os fluxos críticos de ponta a ponta. Para cada um, indique:

1. entrada, autenticação e validação;
2. leitura ou gravação principal;
3. cache, fila ou evento, quando houver;
4. resposta ao cliente;
5. retries, falhas, duplicidades e efeitos posteriores.

Separe o caminho que precisa responder ao usuário do trabalho que pode acontecer em background.

**9 - SD Completo**

Una os diagramas e faça o aprofundamento final nos pontos de atenção. O desenho deve deixar visíveis:

* como escala horizontalmente e remove pontos únicos de falha;
* como trata consistência, disponibilidade e degradação;
* como protege dados e dependências;
* como mede latência, erros, saturação e saúde dos fluxos;
* quais são os principais trade-offs e quando a solução precisaria evoluir.

O resultado não precisa ser perfeito ou definitivo. Ele precisa ser coerente: requisitos levam a estimativas, estimativas revelam riscos e riscos justificam os componentes escolhidos.
