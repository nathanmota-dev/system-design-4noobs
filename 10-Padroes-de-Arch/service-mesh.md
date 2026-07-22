# Service Mesh

Service Mesh é um padrão de arquitetura que adiciona uma camada dedicada à comunicação entre microsserviços. Essa camada centraliza preocupações comuns da comunicação, sem exigir que cada serviço implemente as mesmas funcionalidades:

- **Segurança:** autenticação entre serviços, autorização e *mTLS* (*mutual TLS*).
- **Observabilidade:** métricas, rastreamento distribuído e registros das chamadas.
- **Gerenciamento de tráfego:** roteamento, balanceamento de carga, *retries* e *timeouts*.

### Como funciona

Em geral, o Service Mesh utiliza um proxy associado a cada microsserviço. Esse proxy, que costuma ser executado como um *sidecar*, intercepta o tráfego de entrada e de saída. Assim, os serviços se comunicam por meio dos proxies, que aplicam as políticas do mesh e encaminham as chamadas aos demais serviços.

Esses proxies formam o *data plane*, responsável por lidar diretamente com o tráfego. O *control plane* não costuma participar de cada requisição; ele distribui configurações, políticas e certificados para os proxies.

O papel do *control plane* é:

- Configurar os proxies dos microsserviços.
- Definir regras de roteamento e balanceamento.
- Centralizar políticas de segurança e comunicação.
- Gerenciar certificados e configurações de *mTLS*.

Com essa infraestrutura, os proxies podem:

- Aplicar *retries* e *timeouts*.
- Fazer balanceamento de carga.
- Implementar circuit breaking para evitar que falhas se propaguem.
- Dividir o tráfego para testes A/B, *canary releases* e outras estratégias de implantação.
- Coletar métricas e informações para rastreamento distribuído.

## Quando vale a pena usar

- Muitos microsserviços e equipes diferentes.
- Comunicação interna complexa.
- Necessidade de aplicar políticas de segurança de forma consistente.
- *Deploys* graduais, *canary releases* ou regras de roteamento avançadas.
- Necessidade de centralizar a observabilidade da comunicação entre serviços.

Se o sistema possui poucos microsserviços ou uma comunicação simples, um Service Mesh pode ser mais complexo do que o necessário. Nesse caso, bibliotecas da aplicação ou um API Gateway podem resolver o problema com menor custo operacional.

## Trade-offs

- Maior complexidade operacional e de configuração.
- Mais infraestrutura para implantar, atualizar e monitorar.
- Consumo adicional de CPU e memória pelos proxies.
- Aumento potencial da latência e dificuldade maior para depurar falhas distribuídas.

## Opções

- Istio — Kubernetes e outros ambientes.
- Linkerd — Kubernetes.
- HashiCorp Consul Service Mesh — Kubernetes e máquinas virtuais.
