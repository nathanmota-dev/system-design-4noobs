# Design Anti-Patterns

Aqui estão alguns padrões de arquitetura que devem ser evitados em entrevistas.

O primeiro é o *sharding*. Não que ele seja ruim: se temos um sistema cujo banco de dados precisa lidar com bilhões de escritas, provavelmente precisaremos dele. O ponto é que:

Incluir *sharding* em um sistema aumenta significativamente sua complexidade e sua manutenção. Não basta dizer: “Vou armazenar os IDs dos usuários em diferentes shards”. É preciso considerar que as consultas serão muito mais difíceis do que seriam se o banco de dados estivesse em um único shard. Portanto, uma boa prática é identificar o gargalo e aumentar os requisitos gradualmente, somente quando o entrevistador solicitar.

Exemplo: para escalar um banco de dados, em muitos casos temos mais leituras do que escritas. Nesse cenário, podemos escalar cópias do banco de dados destinadas apenas à leitura, limitar as escritas a uma única máquina e escalá-la verticalmente, aumentando sua capacidade.

Outro exemplo de *anti-pattern* ocorre quando precisamos escrever em dois bancos de dados diferentes, mas fazemos as escritas em uma única transação: por exemplo, enviamos uma escrita para o PostgreSQL e outra para o Redis. Essa não é uma boa prática, pois a transação não é atômica e pode resultar em inconsistências. Uma abordagem melhor é adotar uma arquitetura *event-driven*, adicionar uma fila de mensagens (como Kafka ou RabbitMQ) e publicar eventos que serão consumidos pelos bancos de dados.

Outro exemplo de *anti-pattern* ocorre quando precisamos manter dados sincronizados em bancos de dados de diferentes regiões. Em um sistema distribuído, podemos ter um banco de dados no Brasil, outro nos EUA e outro na Europa, por exemplo. Um padrão ruim nesse caso seria utilizar *locks*, pois bloquearíamos os três bancos de dados em uma única transação. Isso prejudica a escalabilidade, aumenta a fila e exige que os três bancos estejam disponíveis ao mesmo tempo. Quando não precisamos de consistência imediata — ou seja, quando o usuário que acessou o sistema no Brasil precisa ter o login criado no Brasil, mas a criação do login nos EUA e na Europa pode demorar —, devemos optar pela consistência eventual e usar um padrão como o *Saga* para garanti-la.

Outro exemplo ruim é ter apenas um banco de dados principal, mesmo que existam réplicas. Isso caracteriza um SPOF (*Single Point of Failure*, ou ponto único de falha): se o banco de dados principal cair e nenhuma réplica puder assumir seu lugar, todo o sistema poderá cair. O ideal nesse caso é ter réplicas capazes de assumir o papel principal, utilizando um padrão de replicação que garanta a consistência eventual. Em um sistema distribuído global, essa ideia vale para qualquer componente: não queremos ter um único API Gateway, um único Load Balancer etc. Portanto, em entrevistas, é importante ficar atento a esse padrão e considerar implantações em múltiplas regiões e o uso de *geo-routing*.
