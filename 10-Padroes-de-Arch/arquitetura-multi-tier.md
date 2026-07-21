# Arquitetura Multi-Tier

A arquitetura *multi-tier* divide um sistema em camadas (*tiers*) com responsabilidades diferentes. Cada camada concentra um tipo de trabalho e, normalmente, se comunica apenas com as camadas adjacentes.

Um exemplo comum é a arquitetura de três camadas (*three-tier*):

- **Apresentação:** interface com o usuário, como uma aplicação web ou mobile.
- **Negócio:** regras de negócio e processamento das requisições.
- **Dados:** persistência e consulta das informações, geralmente em um banco de dados.

Essas camadas podem estar no mesmo processo ou ser executadas em componentes separados. Por isso, *tier* costuma representar uma separação física ou de implantação, enquanto *layer* representa principalmente uma separação lógica. No uso cotidiano, os dois termos muitas vezes são tratados como sinônimos.

Uma arquitetura multi-tier não exige exatamente três camadas. Dependendo do sistema, podem existir camadas adicionais, como uma camada de serviços, cache ou integração com sistemas externos.

## Vantagens

- **Separação de responsabilidades:** cada camada possui um objetivo mais claro.
- **Manutenção mais simples:** mudanças em uma camada tendem a afetar menos o restante do sistema.
- **Escalabilidade independente:** quando as camadas estão separadas, é possível aumentar a capacidade apenas do componente que se tornou um gargalo.
- **Organização do trabalho:** diferentes times podem trabalhar em partes bem definidas do sistema, desde que os contratos entre elas sejam claros.

## Desvantagens

- **Mais complexidade operacional:** separar as camadas pode exigir deploys, monitoramento e comunicação entre diferentes componentes.
- **Overhead de comunicação:** uma requisição pode atravessar várias camadas e gerar mais latência.
- **Debugging mais difícil:** um problema pode estar na lógica, na comunicação entre camadas ou na infraestrutura.
- **Acoplamento entre contratos:** alterações nas interfaces de uma camada podem exigir mudanças nas demais.

## Relação com outros estilos de arquitetura

Multi-tier descreve principalmente **como as responsabilidades são organizadas**. Outros estilos descrevem aspectos diferentes do sistema e podem ser combinados com essa abordagem.

### Event-Driven

Em uma arquitetura *event-driven*, os componentes se comunicam por meio de eventos. Um produtor publica um evento em um *broker*, e um ou mais consumidores reagem a ele de forma assíncrona.

Esse estilo define a forma de comunicação entre componentes; ele não substitui a divisão em tiers. Por exemplo, a camada de negócio pode publicar um evento que será consumido por uma camada de integração ou por outro serviço.

### Microservices

Microservices organiza o sistema em serviços menores e relativamente independentes, cada um responsável por um domínio ou capacidade de negócio.

Um microserviço pode ter suas próprias camadas de apresentação, negócio e dados. Portanto, microservices e multi-tier podem ser usados juntos, mas representam decisões diferentes: uma define a decomposição em serviços; a outra, a organização das responsabilidades dentro deles.

O objetivo deste documento é apresentar essa visão geral. As características, os benefícios e os trade-offs de Event-Driven, Microservices e outras arquiteturas podem ser aprofundados em documentos específicos.
