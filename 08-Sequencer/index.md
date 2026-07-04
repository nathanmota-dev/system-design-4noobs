# Sequencer / snowwflake

Snowflake é uma técnica que foi criada pelo Twitter para conseguir lidar com o problema de geração de IDs únicos em um ambiente distribuído, onde se a gente tem vários servidores e vários bancos de dados, se cada um deles precisa gerar IDs únicos, pode ser que um servidor gere um id que colida com outro, por isso essa técnica sugiu.

O objetivo dela é garantir 3 coisas
1 - Performance
2 - Nenhum conflito de ID
3 - Algum tipo de ordenação temporal (Não vai garantir uma ordenção 100% em tempo real, mas sim uma ordenação aproximada, pra ordenção em time perfeita é praticamente impossível)

Ela funciona assim:

Primeiro, o algoritmo tem 64 bits, divididos em 4 partes:
- 1 bit para o sign (sinal)
- 41 bits para o timestamp
- 10 bits para o worker ID
- 12 bits para o sequencial

Os bits de timestamp representam o momento em que o ID foi gerado por data, em milissegundos desde a época Unix (1970-01-01 00:00:00 UTC), isso permite que o ID seja ordenado temporalmente. Isso é essencial para garantir que em buscas por tweets em uma ordem de data, exemplo do dia 1 ao dia 4 de um mês, os tweets sejam retornados em ordem cronológica, diminuindo a carga do BD em consultas. O máximo são 2^41 - 1 milissegundos, ou seja, aproximadamente 69 anos.

Os bits de worker ID representam o ID do worker que gerou o ID, e os bits de sequencial representam o número de IDs gerados por worker em um determinado timestamp. Como são 10 bits para o worker ID e 12 bits para o sequencial, o algoritmo pode suportar até 1024 workers e 4096 IDs por worker em um determinado timestamp. Isso significa que a gente poderia ter 1024 servidores e atribuir um id pra cada um, a gente trata como worker e não servidor porque não necessáriamente precisamos de um servidor para cada worker ou utilizar um servidor, poderia ser uma lambda por exemplo.

Os bits sequenciais representam o número de IDs gerados por worker em um determinado timestamp. Quando o sequencial chega a 4096, ele é resetado para 0 e o timestamp é incrementado para o próximo worker.
