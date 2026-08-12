# Segurança

Segurança em System Design é o conjunto de decisões que protege usuários, dados, serviços e infraestrutura contra acesso indevido, alteração, indisponibilidade e abuso. Ela não é apenas uma etapa de autenticação na entrada da aplicação. Cada fronteira entre cliente, gateway, serviço, banco, fila e operador precisa ter uma proteção adequada.

Um sistema seguro começa entendendo o que precisa ser protegido, de quem e com qual impacto caso ocorra uma violação. Depois, os controles são distribuídos pela arquitetura para reduzir a probabilidade e o raio de uma falha.

## Os objetivos de segurança

Os três objetivos clássicos são:

- **Confidencialidade:** somente pessoas e serviços autorizados podem acessar o dado.
- **Integridade:** o dado não pode ser alterado de forma indevida ou silenciosa.
- **Disponibilidade:** o serviço e os dados devem continuar acessíveis para quem tem direito de usá-los.

Autenticidade e rastreabilidade também são importantes: precisamos saber quem está fazendo uma operação e conseguir investigar ações relevantes depois.

Segurança e resiliência se relacionam. Um ataque pode ser uma causa de indisponibilidade, mas aumentar disponibilidade sem controlar acesso pode ampliar o impacto de um abuso.

## Threat modeling

Antes de escolher ferramentas, identifique:

1. quais são os ativos importantes;
2. quem pode tentar atacá-los;
3. quais são as fronteiras de confiança;
4. quais entradas são controladas por terceiros;
5. qual impacto teria cada violação;
6. quais controles reduzem a probabilidade ou o impacto.

Exemplos de ativos:

- credenciais e tokens;
- dados pessoais;
- dados financeiros;
- pedidos e saldos;
- chaves de API;
- código e imagens de deploy;
- logs e dados de auditoria.

O threat model deve considerar usuários mal-intencionados, contas comprometidas, serviços internos vulneráveis, operadores com excesso de acesso e dependências de terceiros.

## Autenticação e autorização

Autenticação responde: **quem é você?**

Autorização responde: **o que você pode fazer?**

Um usuário autenticado não deve automaticamente acessar todos os recursos. O serviço precisa validar a permissão para a ação e o recurso específico.

O módulo de [Autenticação e Autorização](../06-Auth/index.md) aprofunda esses conceitos.

### Sessões e tokens

- **Sessão no servidor:** o cliente recebe um identificador; o estado da sessão fica armazenado no servidor ou em um armazenamento compartilhado.
- **Token assinado:** o cliente envia um token que contém informações verificáveis, como identidade e expiração.
- **OAuth/OIDC:** permite delegar autenticação e autorização entre aplicações e provedores de identidade.

Sessões facilitam revogação imediata, mas exigem armazenamento e compartilhamento de estado. Tokens podem ser validados localmente e escalar melhor, mas revogar um token antes da expiração exige uma estratégia adicional, como expiração curta, rotação ou lista de revogação.

### RBAC e ABAC

- **RBAC:** permissões são associadas a papéis, como `admin`, `editor` e `viewer`.
- **ABAC:** a decisão considera atributos do usuário, do recurso e do contexto, como tenant, região, horário e classificação do dado.

RBAC é simples de entender e operar, mas pode gerar muitos papéis. ABAC é mais expressivo, mas aumenta a complexidade de políticas, testes e auditoria.

## Princípio do menor privilégio

Cada usuário, serviço ou processo deve possuir somente as permissões necessárias para executar sua função.

Na prática:

- o serviço de pedidos não precisa ser administrador do banco;
- um worker de leitura não precisa de permissão de escrita;
- um pipeline de deploy não precisa acessar todos os dados de produção;
- uma aplicação não deveria usar a mesma credencial em todos os ambientes;
- acessos administrativos devem ser temporários e auditáveis.

O menor privilégio limita o impacto de uma credencial vazada ou de uma vulnerabilidade. O custo é mais trabalho para definir, testar e manter permissões corretas.

## Proteção de dados

### Em trânsito

Use TLS para proteger a comunicação entre cliente e borda e, quando necessário, também entre serviços internos. Uma rede interna não deve ser considerada confiável apenas por estar dentro da infraestrutura.

### Em repouso

Bancos, backups, objetos e filas podem conter informações sensíveis. Criptografia em repouso reduz o impacto de acesso ao armazenamento, mas não protege um serviço que já possui permissão de leitura.

### Chaves

As chaves devem ser armazenadas e rotacionadas por um mecanismo apropriado de secrets management, e não em código, imagem de container ou repositório.

A rotação precisa considerar versões antigas, rollback e serviços que ainda estão usando a chave anterior. Rotacionar sem planejar a transição pode causar indisponibilidade.

### Minimização e retenção

Não armazene um dado apenas porque é possível. Colete o mínimo necessário, defina retenção e elimine dados quando eles não forem mais necessários.

Logs e traces também podem carregar informações pessoais. Mascaramento, tokenização e controle de acesso são necessários nesses sistemas auxiliares.

## Segurança da API

Na borda da aplicação, controles comuns incluem:

- autenticação e autorização;
- validação de schema, tamanho e tipo de entrada;
- rate limiting e proteção contra abuso;
- limites de payload e timeout;
- proteção contra replay quando a operação for sensível;
- versionamento de contratos;
- tratamento seguro de erros;
- registro de ações relevantes para auditoria.

O API Gateway pode centralizar controles comuns, mas não deve ser a única camada de autorização. O serviço de domínio ainda precisa validar se o usuário pode executar aquela operação sobre aquele recurso.

Mais detalhes sobre a função desse componente estão em [API Gateway](../04-API-Gateway/01-API-Gateway.md).

## Isolamento de rede

A arquitetura pode separar componentes em redes, sub-redes e grupos de segurança diferentes. Um banco não deveria ser exposto diretamente à internet se apenas a aplicação precisa acessá-lo.

Um desenho comum é:

```text
Internet
   │
   ▼
CDN / WAF / Load Balancer
   │
   ▼
API ou serviços de aplicação
   │
   ├── Banco de dados privado
   ├── Cache privado
   └── Filas e workers privados
```

O isolamento reduz a superfície de ataque, mas não substitui autenticação e autorização. Uma vez que um invasor obtenha acesso a uma rede interna, os serviços ainda precisam verificar a identidade e as permissões uns dos outros.

## Segurança entre serviços

Em uma arquitetura distribuída, serviços precisam autenticar chamadas internas e controlar permissões. Algumas opções são:

- tokens de serviço;
- certificados e mTLS;
- identidade de workload;
- políticas por serviço e por operação;
- rotação automática de credenciais.

mTLS pode oferecer autenticação forte e criptografia entre serviços, mas aumenta a complexidade de certificados, rotação e diagnóstico. Um token simples pode ser suficiente em um ambiente menor, desde que seja protegido e tenha escopo limitado.

## Auditoria e detecção

Logs de auditoria registram ações relevantes, como:

- alteração de permissões;
- acesso a dados sensíveis;
- criação ou cancelamento de pedidos;
- mudança de configuração;
- login, falha de autenticação e revogação de sessão;
- operação administrativa.

Um log de auditoria precisa ser protegido contra alteração e ter contexto suficiente para investigação. Não deve registrar segredos ou payloads sensíveis sem necessidade.

Além de prevenir, o sistema deve detectar comportamentos anormais, como volume incomum de tentativas, acesso a muitos recursos ou uso de uma credencial em regiões inesperadas.

## Multi-tenant

Em sistemas com vários clientes ou organizações, o tenant precisa ser considerado em todas as camadas:

- identificação do tenant autenticado;
- autorização da operação;
- filtro obrigatório nas consultas;
- chave de particionamento;
- cache isolado por tenant;
- logs e métricas sem vazamento entre clientes;
- backup e exportação com escopo correto.

Um filtro de tenant esquecido em uma única consulta pode expor dados de outra organização. Por isso, isolamento deve ser reforçado por políticas, testes automatizados e revisão de acesso.

## Segurança de dependências e deploy

A cadeia de desenvolvimento também faz parte do sistema:

- valide dependências e imagens;
- aplique atualizações de segurança;
- limite quem pode publicar artefatos;
- assine ou verifique imagens quando necessário;
- não coloque secrets em logs de pipeline;
- separe ambientes e permissões;
- tenha rollback para mudanças de configuração.

Uma vulnerabilidade em uma biblioteca ou imagem pode atingir todas as instâncias. Segurança não termina quando o código chega ao ambiente de produção.

## Exemplo: API de dados pessoais

Considere uma API que permite consultar dados pessoais:

```text
Cliente → Gateway/WAF → Serviço de Identidade
                    └→ Serviço de Dados → Banco criptografado
                                           └→ Auditoria
```

Uma possível sequência é:

1. o cliente estabelece conexão TLS;
2. o gateway aplica limite e valida o token;
3. o serviço de dados verifica a autorização sobre o recurso e o tenant;
4. a consulta usa uma identidade com menor privilégio;
5. os dados retornados são minimizados para a finalidade da operação;
6. o acesso relevante gera um evento de auditoria sem registrar o dado completo;
7. métricas e alertas detectam volume anormal ou falhas de autenticação.

O gateway ajuda a bloquear tráfego inválido, mas a decisão de negócio continua no serviço. O banco pode estar criptografado, mas isso não elimina a necessidade de controlar quem possui permissão de leitura.

## Trade-offs

- **Segurança x usabilidade:** MFA, expiração curta e verificações adicionais reduzem risco, mas adicionam fricção.
- **Segurança x latência:** criptografia, inspeção e chamadas a um provedor de identidade adicionam custo de tempo.
- **Menor privilégio x agilidade:** permissões restritas reduzem impacto, mas exigem modelagem e manutenção.
- **Tokens stateless x revogação:** validação local escala bem, mas revogação imediata é mais difícil.
- **Retenção de logs x privacidade:** mais dados facilitam investigação, mas aumentam risco, custo e obrigações de proteção.
- **Isolamento x complexidade:** redes e contas separadas reduzem o raio de impacto, mas tornam deploy e operação mais complexos.
- **Disponibilidade x controle:** depender de um provedor de identidade para cada operação pode aumentar segurança, mas pode tornar esse provedor um ponto crítico de falha.

O objetivo não é tornar toda operação impossível; é aplicar controles proporcionais ao risco e ao valor do ativo protegido.

## Como aplicar em uma entrevista de System Design

Inclua perguntas e decisões como:

1. Quais são os ativos e os dados sensíveis?
2. Quem são os atores e quais são as fronteiras de confiança?
3. Como autenticar usuários e serviços?
4. Como autorizar acesso ao recurso específico?
5. Onde aplicar TLS, rate limiting, validação e isolamento de rede?
6. Como armazenar, rotacionar e revogar secrets?
7. Como impedir vazamento entre tenants?
8. Quais ações precisam de auditoria e alertas?
9. Como responder se uma credencial for comprometida?
10. Qual impacto os controles terão em latência, disponibilidade e operação?

Uma resposta madura explica o risco que cada controle reduz e o trade-off que ele introduz.

## Resumo

Segurança deve ser parte do desenho desde o início. Autenticação, autorização, menor privilégio, criptografia, gestão de secrets, isolamento de rede, auditoria, proteção de APIs e segurança da cadeia de deploy trabalham juntos.

O trade-off central é equilibrar proteção, custo, desempenho, disponibilidade e experiência do usuário. O melhor design não é o que adiciona controles indiscriminadamente, mas o que protege os ativos certos em cada fronteira do sistema.
