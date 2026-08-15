# Federação de bancos de dados

Federação é uma estratégia que permite dividir dados entre bancos independentes e oferecer uma camada capaz de consultar ou rotear operações entre eles. A divisão pode ocorrer por região, domínio ou requisito regulatório.

Na prática, a camada de federação pode atuar como:

- Roteador.
- Agregador.
- Coordenador de queries.

Ela não sincroniza os bancos automaticamente; sua responsabilidade principal é localizar e combinar os dados. Dependendo do desenho, pode:

- Decidir qual banco ou região recebe uma escrita.
- Definir quais regiões participam de uma query.
- Combinar resultados de diferentes regiões em uma única resposta.

Além da distribuição geográfica, a federação pode atender a requisitos de residência de dados. Certos dados podem precisar permanecer em países ou regiões específicos por questões regulatórias e contratuais.

Para evitar consultar todas as regiões, um diretório pode mapear o identificador do dado para sua localização:

| `user_id` | Região |
|---|---|
| `nathan` | Europa |

Esse índice melhora o desempenho do roteamento, mas também precisa ser mantido consistente quando um dado muda de região.

## Como federação é abordada em entrevistas de System Design

A forma mais simples de abordar federação em uma entrevista é explicar como localizar a região responsável pelo usuário. Em vez de verificar todos os bancos, o servidor consulta um diretório com o `user_id` e a região correspondente e redireciona a operação para o banco correto.

Em um sistema com muito mais leituras do que escritas, também pode fazer sentido manter réplicas de leitura em cada região. O servidor lê da réplica mais próxima, mas direciona a escrita para a região responsável pelo dado. O trade-off é lidar com a latência de escrita e com o atraso da replicação.

Uma *federated query layer* consulta fontes diferentes e combina seus resultados. Essa estratégia pode facilitar consultas globais, mas adiciona latência, custo e complexidade, principalmente em joins entre regiões. Em uma entrevista, vale citá-la somente quando existe uma necessidade clara de consulta federada.

---
