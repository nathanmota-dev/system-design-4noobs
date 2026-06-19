## Federation

Federation é uma estratégia de banco de dados que foi criada pra conseguir lidar com bancos de dados em diferentes regiões onde ela permite dividir um banco de dados em várias partes (federations) e consultar todas elas de forma transparente.

Na prática o Federation pode ser:

- Router
- Agregador
- Coordernador de Queries

Ele funciona como uma peça que nos ajuda na sincronização de dados entre diferentes bancos de dados que estão em diferentes regiões. Ele tem algumas resposabilidades e dependendo do que a gente quer que ele faça, ele pode ser configurado de diferentes maneiras. As principais responsabilidades do Federation incluem:

- Qual DB regiao recebe uma escrita
- Quais regioes participam de uma query
- Combinar os resultados de diferentes regioes em uma única resposta

Além de poder ter dados em diferentes regiões como foi citado, ele pode ser usado caso a gente tenha que dividir os dados por questões regulatórias de países, exemplo alguns países da europa não permitem armazenar tipos de dados sensíveis em uma região específica, pra isso o Federation serve para dividir os dados em diferentes regiões e garantir que os dados sensíveis sejam armazenados em regiões seguras.

Quando dados são muito usados, eles podem ser armazenados em um tabelão que inclui o dado e a região onde ele está armazenado como:

user_id | region
nathan  | europe

melhorando significativamente a performance das consultas distribuídas entre as regiões.

### Como federation e pedido em entrevistas de SD

A forma mais simples que Federation e pedido em entrevistas de SD e seguindo um exemplo anterior onde vamos pensar que a gente tem um user e ele precisa fazer uma escrita de dados como adicionar um usuario, entao o servidor primeiro verifica o cache e o banco de dados daquela região, se não encontrar, ele verifica as outras regiões e se encontrar, ele retorna o resultado. Normalmente como foi falado anteiormente pra diminuir esse tempo de resposta se essa operacao for muito comum, como por exemplo no facebook e, pra nao verificar todas as regiões, teria uma tabela com o user_id e a região onde ele está armazenado, assim quando o servidor precisa fazer uma escrita, ele verifica essa tabela e redireciona para a região correta.

Outra estratégia seria, vamos supor na netflix onde e um sistema que a gente tem muito mais leituras do que escritas, nesse caso faria mais sentido fazer multiplas replicas de leitura para cada região, assim quando o servidor precisa fazer uma leitura, ele pode escolher qualquer uma das replicas disponiveis, aumentando a disponibilidade e a performance do sistema porém quando ele precisar fazer uma escrita vai ter uma latência maior porque ele precisa escrever na região específica onde o dado está armazenado.

Federated query layer - É uma api em camadas que permite a leitura de dados em diferentes regiões e armazenar todos em joins em uma única tabela diminuindo siginficativamente. Essa estrátégia é bem complexa e custosa então não é sempre que vale a pela e é recomendado apenas para casos de leitura muito comum, normalmente vale no máximo citar mas não é uma solução muito comum.

---
