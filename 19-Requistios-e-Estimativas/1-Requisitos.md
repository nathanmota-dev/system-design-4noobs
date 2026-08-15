# Requisitos

Para montar um System Design, o primeiro passo é entender o problema e seus requisitos.

Por exemplo, vamos supor que foi proposto para a gente o seguinte problema: encurtar uma URL.

Para isso, a gente precisa:

- Criar uma URL encurtada
- Acessar a URL longa baseado na URL encurtada

Com base nisso, podemos pensar em outras coisas:

- Para que a gente quer essa URL curta? Precisamos de analytics, contador de cliques ou visitantes únicos?
- Os Links expiram?
- Essa URL precisa ser customizada? Se o usuário puder escolher o identificador, a gente precisará verificar se ele já existe.
- A URL será gerada com ID aleatório? Se sim, qual método será usado?

Esses seriam requisitos funcionais. Também precisamos levantar requisitos não funcionais:

- Quantos usuários?
- Qual latência?
- Qual disponibilidade?
- Qual SLA?

Vamos supor que depois de fazer todas essas perguntas sobre requisitos funcionais e requisitos específicos foi informado:

- 10 anos de retenção (URLs precisam durar 10 anos pelo menos)
- URL customizada pelo usuário
- Analytics básico
- 99.99% de uptime
- Latência de até 50ms (P95 < 50ms)  

Também foi informado que o sistema deve suportar:

- 500 milhões de usuários
- Média de 2 URLs por usuário
- 1,2 bilhão de novas URLs por ano
- 12 bilhões de URLs
- 1,2 trilhão de leituras por dia

Com base nesses números, podemos fazer estimativas de vazão média. Esse cálculo é uma divisão por tempo, não a Lei de Little, que relaciona quantidade no sistema, taxa de chegada e tempo médio.

Escrita:

- 1,2 bilhão de escritas por ano ÷ 365 ≈ 3,29 milhões por dia
- 3,29 milhões por dia ÷ 24 ≈ 137 mil por hora
- 137 mil por hora ÷ 60 ≈ 2.283 por minuto
- 2.283 por minuto ÷ 60 ≈ 38 escritas por segundo

Leitura:

- 1,2 trilhão de leituras por dia
- 50 bilhões por hora
- Aproximadamente 13,9 milhões por segundo

Com base nesses números, podemos estimar que o gargalo principal do nosso sistema é a leitura, pois é ela que mais consome recursos, ou seja, nossa escala tem que ser nela.

---

Agora vamos pensar em como atingir 99,99% de disponibilidade. Para uma estimativa real, é necessário consultar o SLA de cada serviço e entender se os componentes estão em série ou têm redundância.

Se uma EC2 e um RDS forem ambos necessários e cada um tiver disponibilidade de 99,99%, uma aproximação com falhas independentes seria `0,9999 × 0,9999 = 0,99980001`, ou cerca de 99,98%. A disponibilidade de componentes em série é multiplicada, não somada nem calculada por média. Réplicas, múltiplas zonas e failover podem melhorar o resultado, mas o cálculo depende da independência das falhas e do tempo de recuperação.

---

Normalmente, em entrevistas, não precisamos calcular tudo com precisão. A ideia é mostrar quais números orientam a escala do sistema e declarar claramente as premissas usadas.
