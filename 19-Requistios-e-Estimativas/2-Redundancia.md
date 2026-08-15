# Redundância

## Duplicação de componentes

A primeira coisa a entender sobre redundância é que um componente único pode se tornar um ponto único de falha. Se o sistema possui apenas um banco e ele fica indisponível, as operações que dependem dele também ficam. Uma réplica em outra zona pode permitir failover, enquanto um backup permite recuperar dados; são soluções diferentes e as duas podem ser necessárias.

## Health check

Health Check é um processo que verifica se os componentes do sistema estão funcionando corretamente.

## SLA

SLA (*Service Level Agreement*) é um acordo entre o provedor e o cliente que define níveis de serviço, como disponibilidade e desempenho, e pode prever consequências quando não são cumpridos. Entender o SLA necessário é fundamental, porque cada nove adicional reduz bastante a margem anual de indisponibilidade:

- 99% ≈ 3 dias e 15 horas.
- 99,9% ≈ 8 horas e 46 minutos.
- 99,99% ≈ 52 minutos e 34 segundos.
- 99,999% ≈ 5 minutos e 15 segundos.

A estratégia para atingir a meta pode usar múltiplas instâncias e zonas de disponibilidade. Múltiplas regiões são necessárias apenas quando os requisitos e modos de falha justificam o custo e a complexidade. Também é preciso considerar deploys, dependências, dados, monitoramento e tempo de failover; replicar um componente não garante sozinho a disponibilidade do sistema inteiro.
