# Sequencer e Snowflake IDs

Snowflake é uma técnica criada pelo Twitter para gerar IDs únicos em um ambiente distribuído. Quando vários servidores geram identificadores de forma independente, pode acontecer uma colisão; o Snowflake evita esse problema sem depender de um contador central a cada operação.

O objetivo é garantir três características:

1. Alto desempenho.
2. Ausência de conflito entre IDs, desde que cada worker tenha um identificador único.
3. Ordenação temporal aproximada. O ID não garante uma ordem global perfeita quando existem diferenças entre os relógios das máquinas.

Ela funciona assim:

O algoritmo usa 64 bits, divididos em quatro partes:

| Bits | Campo | Função |
|---:|---|---|
| 1 | Sinal | Mantém o ID positivo em uma representação com sinal. |
| 41 | Timestamp | Representa o tempo desde uma época definida pela implementação. |
| 10 | Worker ID | Identifica o worker que gerou o ID. |
| 12 | Sequência | Diferencia IDs gerados pelo mesmo worker no mesmo milissegundo. |

Os bits de timestamp representam, em milissegundos, o tempo desde uma época definida pela implementação. Isso permite ordenar os IDs aproximadamente pelo momento de criação e melhora a localidade em índices. Com 41 bits, o intervalo disponível é de aproximadamente 69 anos.

Com 10 bits para o worker ID e 12 bits para a sequência, essa configuração suporta até 1.024 workers e 4.096 IDs por worker em cada milissegundo. Um worker não precisa corresponder diretamente a um servidor físico, mas seu identificador não pode colidir com o de outro gerador ativo.

Quando um worker esgota as 4.096 combinações no mesmo milissegundo, ele precisa esperar o próximo milissegundo antes de reiniciar a sequência. Outro cuidado importante é o *clock rollback*: se o relógio da máquina voltar no tempo, a implementação precisa esperar, rejeitar a geração ou usar outra estratégia para não produzir IDs repetidos.
