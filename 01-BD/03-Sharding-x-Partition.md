## Sharding:

Basicamente, em BDs com grande escala, é comum dividir os dados com o objetivo de diminuir a latência, melhorar o tempo de consulta, leitura e escrita e distribuir melhor a carga.

Dentro dessa divisão, podemos pensar em duas abordagens:

1 - Particionamento vertical
2 - Particionamento horizontal

No particionamento vertical, as colunas ou tabelas do BD são divididas em múltiplos bancos de dados de acordo com a responsabilidade. Exemplo: assim como em microserviços, em que cada microserviço pode ter seu próprio DB cuidando de apenas uma parte do domínio, como um BD somente para users, a lógica é parecida: cada BD passa a cuidar de apenas uma coisa.
Outra forma de particionar verticalmente é separar a responsabilidade por colunas. Vamos supor que estamos usando 3 bancos de dados: os 3 podem ter tabelas relacionadas ao mesmo contexto, mas com colunas diferentes em cada um.
Pontos negativos do particionamento vertical: cria uma camada grande de complexidade caso seja necessário fazer joins em consultas, ou seja, aumenta consideravelmente a complexidade da aplicação.
Pontos positivos: separação de responsabilidades e possibilidade de partes diferentes dos dados escalarem de forma independente.

Já no particionamento horizontal, a ideia é separar as linhas em diferentes bancos de dados. Exemplo: a linha com `user_id = 1` fica no banco de dados 1, a linha com `user_id = 2` fica em outro banco de dados, e assim vai.
Quando esse particionamento horizontal é distribuído entre múltiplos bancos, normalmente é isso que chamamos de sharding.
Pontos negativos disso: quando precisamos fazer uma consulta, precisamos de alguma técnica para localizar os dados e saber em qual banco estará o id daquele user. Para isso, existem 3 formas comuns:

1 - Key-range

No key-range, você delimita a faixa de ids que cada banco vai armazenar. Por exemplo: no banco 1 ficam os ids de 1 a 10.000; no banco 2, de 10.001 a 20.000; e no banco 3, de 20.001 a 30.000.
Ponto negativo disso: vamos supor que o app escale e passe de 30.000 ids. Se você redefinir essas faixas, pode precisar realocar muitos dados. Por exemplo, se agora o BD 1 vai de 0 a 100.000, o 2 de 100.001 a 200.000 e o 3 de 200.001 a 300.000, será necessário mover vários ids antigos para a nova faixa correta.

2 - Módulo

Na estratégia de módulo, funciona assim:

Você pega o id do usuário, calcula o módulo pela quantidade de BDs distribuídos e usa esse resultado para definir onde o dado será alocado. Exemplo: `id % quantidade_de_bancos`.
Ponto negativo: se você acrescentar um banco a mais, por exemplo, dados distribuídos em 3 bancos e depois em 4, a conta do módulo muda e você vai precisar realocar os dados.

3 - Consistent hashing

O consistent hashing surgiu para melhorar o rebalanceamento dos dados. Nessa estratégia, tanto os dados quanto os bancos são posicionados em um espaço de hash, e cada dado é alocado no primeiro nó responsável a partir do hash gerado para aquela chave.
Exemplo simplificado: vamos supor que temos o `db1` em 250.000, o `db2` em 500.000, o `db3` em 750.000 e o `db4` em 1.000.000. Se o hash de um dado for 900.000, ele ficaria alocado no `db4`.
Outro ponto: ele facilita o rebalanceamento porque, se precisarmos acrescentar um novo banco de dados, por exemplo na faixa de 125.000, em vez de alterar todos os dados, normalmente movemos apenas os dados daquela faixa que passaram a pertencer ao novo nó. Isso reduz bastante a quantidade de realocação.

---

## Partitions

Partitions seguem uma ideia parecida no sentido de dividir os dados em múltiplas partes, mas normalmente essa separação é lógica dentro do próprio banco ou da própria tabela. Dependendo da tecnologia, isso pode ser feito por faixa, hash ou outras regras.
De forma simplificada, podemos pensar em duas formas de trabalhar com partições:

- Particionamento dinâmico
- Particionamento estático

No particionamento estático, as partições e seus limites são definidos antecipadamente.
No particionamento dinâmico, esse particionamento também é definido, mas pode ser ajustado no futuro com realocação dos dados.

Um grande desafio com partitions é buscar dados que não usam a chave primária. Exemplo: como cada dado está em uma partição, normalmente é mais fácil encontrar os ids quando eles seguem a regra usada no particionamento, como um range específico. Já buscas por outros campos podem exigir consultar várias partições.

Problemas frequentes:

O problema é: vamos imaginar o Facebook com uma tabela de users, e esses users podem ser buscados por nome no search. Como esses users são buscados frequentemente, precisamos criar um índice secundário global na coluna `name` para conseguir buscar usuários por nome sem depender apenas do id.

Outro problema é: pense que temos os posts do Facebook divididos em múltiplos bancos de dados ou partições por id. Quanto mais antigo for o post, menor tende a ser a frequência de acesso. Normalmente, os posts recentes são mais acessados, ou seja, o banco de dados que contém dados antigos quase não é acessado, enquanto o que contém os ids mais novos é muito acessado.
Para isso, existe uma estratégia chamada cold storage, em que o armazenamento antigo é arquivado em uma camada mais barata e mais lenta de acessar. Isso é feito em aplicações muito grandes para reduzir custo, e normalmente esse cold storage representa boa parte do armazenamento total do app. O Facebook, por exemplo, existe desde 2004, e conteúdos antigos ainda podem precisar ser armazenados até hoje.
Dessa forma, conseguimos deixar os BDs mais quentes com conteúdos recentes e devidamente balanceados, enquanto conteúdos antigos ficam arquivados e as queries para acessá-los tendem a demorar mais.

---

## O que é o Apache ZooKeeper? E como ele é importante nesse cenário que vimos acima?

Basicamente esse serviço é como se fosse um zelador de serviços distribuídos, através dele é possível gerenciar múltiplos micro-serviços, aplicações distribuídas facilitando a comunicação entre os serviços e todo esse gerenciamento. Ele basicamente expõe uma interface simples (parecida com um sistema de arquivos de pastas e arquivos, chamados de znodes) para resolver os problemas mais complexos de sistemas distribuídos:

- Gerenciamento de Configuração: Se você tem 50 instâncias de um serviço rodando e precisa mudar uma variável de ambiente, você muda no ZooKeeper. Ele avisa e atualiza todas as instâncias em tempo real.
- Sincronização Distribuída (Locks): Garante que duas máquinas não tentem executar a mesma tarefa crítica exatamente ao mesmo tempo, evitando corrupção de dados.
- Eleição de Líder (Leader Election): Se o servidor principal (Leader) de um sistema cair, o ZooKeeper coordena os nós restantes (Followers) para eleger um novo líder de forma limpa e automática.
- Service Discovery: Ajuda os nós a descobrirem quais outros servidores estão ativos e prontos para receber requisições na rede.

Como ele é o cérebro da coordenação, ele não pode cair. Por isso, o ZooKeeper roda em um cluster de servidores (chamado de Ensemble).

--- 

## Teorema de CAP

- Consistency
- Availability
- Partition Tolerance

O teorema de CAP afirma que é impossível trabalhar com Sistemas Distribuídos e ter as 3 coisas acima ao mesmo tempo que são Consistência, Alta Dispobilidade e Tolerancia a Partições, caso seu sistema tenha dois desse, ele não vai ter o terceiro. 

---