# Requisitos

Para montar um System Design o primeiro passo é enteder o Problema e seus requisitos.

Por exemplo, vamos supor que foi proposto pra gente um problema que é: Encurtar uma URL.

Pra isso a gente precisa:
- Criar uma URL encurtada
- Acessar a URL longa baseado na URL encurtada

Com base nisso podemos pensar em outras coisas como:
- Pra que a gente quer essa Url curta? Adicionar Analytics? Click counter? Unique visitors?
- Os Links expiram?
- Essa url precisa ser customizada, ou seja, o usuário pode escolher a URL encurtada que ele quiser, porque se sim, a gente vai precisar acessar o banco e verificar se aquela url não existe
- A url vai ser gerada com id aleatório? Se sim, qual método a gente vai usar para gerar o id?

Esses seriam requisitos mais gerais do sistema, porém precisamos levantar também os requisitos técnicos como:
- Quantos usuários?
- Qual Latência?
- Uptime?
- SLA?

Vamos supor que depois de fazer todas essas perguntas sobre requisitos funcionais e requisitos específicos foi informado:

- 10 anos de retenção (URLs precisam durar 10 anos pelo menos)
- URL customizada pelo usuário
- Analytics básico
- 99.99% de uptime
- Latência de até 50ms (P95 < 50ms)  

E foi informado também que o sistema deve suportar 
- 500 milhões de usuários
- 2 URLs média pra cada usuário
- 1.2 bilhão por ano
- 12 bilhões de URLs
- 1.2 trilhões de reads por dia

Com base nesses números, podemos estimar que o sistema precisa suportar criando uma Little's Law usando uma calculadora mesmo

Escrita:

- 1.2 bilhões de reads por dia / 24 = 3.28 MI por dia
- 130k por hora
- 2283 por minuto
- 24 - Writes por segudo

Leitura:

- 1.2 trilhões de reads por dia
- 500 MI por hora
- 13.8 MI por segundo 

Com base nesses números, podemos estimar que o gargalo principal do nosso sistema é a leitura, pois é ela que mais consome recursos, ou seja, nossa escala tem que ser nela.

---

Agora vamos pensar como conseguir ter 99.99% de uptime, ou seja, 99.99% das vezes o sistema deve estar disponível. Vamos supor que a AWS uma ec2 forneça esses 99.99% de uptime, provavelmente é menor mas a gente não precisa saber isso de cabeça e vamos supor que a gente use um RDS pra BD e que esse RDS também tenha 99.99% de uptime.

Somando o Uptime de ambas as partes, se a gente tiver apenas 1 EC2 e 1 RDS, o uptime total seria de 1 EC2 * 99.99% + 1 RDS * 99.99% / 2 = 99.98%, ou seja, não iriamos conseguir bater o uptime desejado, a ideia então é  que a gente vai precisar de replicação de Instâncias e Banco de Dados para conseguir bater o uptime desejado.

---

Normalmente em entrevistas não precisamos calcular o Little's Law, nem esse uptime, a ideia aqui foi entender que precisamos saber algumas coisas para conseguir escalar o sistema e atender os requisitos desejados.