# Redundância

### Duplicação de Componentes

Primeira coisa que a gente tem que entender em Redundância é que se a gente monta um sistema com apenas um Banco de Dados, se esse banco de dados cair, todo o sistema cairá, então a gente precisa de no mínimo uma cópia de segurança.

### Health Check

Health Check é um processo que verifica se os componentes do sistema estão funcionando corretamente.

### SLA

SLA (Service Level Agreement) é um contrato entre o provedor de serviços e o cliente, definindo os níveis de disponibilidade e performance esperados. Pra lembrar isso de forma fácil, SLA é quando a gente contrata uma EC2 onde a AWS fala que a gente vai ter 99% de disponibilidade, isso significa que em 1 ano, esse sistema pode ficar offline no máximo 3 dias e 15 horas no agregado. Entender qual SLA é necessário na hora de montar um System Design é fundamental para garantir a disponibilidade do sistema, porque de 99% de disponilbidade pra 99.99% é uma diferença absurda onde:

- 99% = 3 dias e 15 horas
- 99.9% = 8 horas
- 99.99% = 1 hora
- 99.999% = 5 minutos

Ou seja, com 99% de uptime, uma cloud como AWS já garantirá isso, mas com 99.99% de uptime, a diferença é absurda, a gente vai precisar de diferentes cópias em diferentes regiões pra garantir esse uptime.