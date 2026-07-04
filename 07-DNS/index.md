# DNS - Domain Name System

O DNS (Domain Name System) é um sistema de nomes de domínio que traduz nomes de domínio em endereços IP e deixa essa lista atualizada caso haja alterações nos registros.

Na prática funciona assim, quando a gente acessa o youtube.com e a gente visualiza o site do YouTube, o DNS resolve o nome de domínio para o endereço IP correspondente e a gente vê o site. Isso acontece porque seria muito dificil da gente memorizar o endereço IP do YouTube e ter que digitar manualmente toda vez que quiser acessar, por exemplo um dos endereços do google é 142.250.185.206, que seria completamente inviável, além que esse endereço IP pode mudar caso o servidor do google mude.

### Vamos entender o DNS

O DNS é uma infraestrutura hierárquica, que pode ser recursiva ou iterativa.
- **Recursiva**: o servidor DNS que recebe a requisição do cliente, envia para o servidor TLD ou TOP LEVEL DOMAIN e envia para o servidor DNS do domínio específico que autoriza o acesso ao site e retorna o endereço IP correspondente e depois isso volta recursivamente.
- **Iterativa**: o servidor DNS que recebe a requisição do cliente, redireciona a requisição para o Root Server que retorna a requisiçao, depois ele envia uma requisição pro servidor TLD ou TOP LEVEL DOMAIN e depois para o servidor DNS do domínio específico que autoriza o acesso ao site e retorna o endereço IP correspondente.

### 8 passos no DNS

1. o Usuário digita youtube.com no browser. Query vai para o servidor DNS do browser.
2. Resolvedor busca DNS root nameserver.
3. Root responde com o endereço do TLD server
4. Resolvedor faz request pro servidor TLD.
5. TLD responde com o endereço do servidor DNS do domínio específico.
6. Resolvedor faz request pro servidor DNS do domínio específico.
7. Autoritativo Responde
8. Resolvedor retorna o endereço IP correspondente ao domínio solicitado.

### Cache

O DNS cache é uma técnica utilizada para armazenar em cache os resultados de consultas DNS, para evitar consultas repetidas e acelerar o processo de resolução de nomes. Existem alguns tipos de cache:

- Cache do browser
- Cache do OS
- Cache do DNS resolver
- Cache do ISP

e SOMENTE ai, se eu não achar o endereço vou fazer o fluxo acima de 8 passos, com request pra estrutura do DNS porque não precisa. Isso acontece para diminuir a latência e acelerar o acesso aos sites.

### Endereços de IP

- Os endereços de IP são números que identificam cada dispositivo conectado a uma rede. Existem dois tipos de endereços de IP: IPv4 e IPv6.
  - **IPv4**: endereços de 32 bits, representados em notação decimal (ex: 192.168.1.1).
  - **IPv6**: endereços de 128 bits, representados em notação hexadecimal (ex: 2001:0db8:85a3:0000:0000:8a2e:0370:7334).
- O IPv6 é uma versão mais avançada do IPv4, por causa de uma limitação no número de endereços disponíveis em IPv4.
- Os endereços de IP podem ser públicos (acessíveis pela internet) ou privados (usados internamente em redes locais).
- IOT está bem ligaddo com IPv6 porque vc pode ter dispositivos em uma rede local e querer exibir seus endereços de IP na internet.
- Não necessarimente todo site está hospeado em um servidor IP diferente, isso ocorre porque algumas clouds podem provisionar vários domínios em um único servidor IP como a Vercel, porém separado eles pelo HOST, usando um virtual hosting.
- Ips privados podem repetidos, exemplo eu posso ter uma impressora conectada na minha casa com um ip e outra pessoa ter um ip com o mesmo número, mas em uma rede diferente de outro dispositivo.
- Ips podem ser estáticos ou dinâmicos, dependendo da configuração do servidor.

### IPS Estáticos

- Ips estáticos são endereços de IP que não mudam, eles são configurados manualmente e não são atribuídos dinamicamente pelo servidor. 
- Normalmente eles são mais usados em Infra, Banco de Dados e outros serviços que não precisam de IPs dinâmicos.
- A vantagem do estático é que ele não muda, então é mais fácil de configurar e gerenciar.

### IPS Dinâmicos

- Ips dinâmicos são endereços de IP que mudam automaticamente, eles são atribuídos dinamicamente pelo servidor. 
- Normalmente eles são mais usados em redes internas.
- A vantagem do dinâmico é que ele da mais flexibilidade. 

É interessante se atentar que assim como em uma casa eu posso ter dispostivos que não são acessiveis a internet, ou seja, o endereço de IP é privado e não é acessível pela internet eu posso utilizar essa mesma estrutura para armazenar uma API, onde eu teria apenas o Load Balancer podendo receber requisições de qualquer lugar e distribuí-las para os servidores da API, onde esses servidores não tem acesso direto à internet, nem os bancos de dados e nem as Lambdas por exemplo, mas apenas os serviços que estão dentro da infraestrutura da API.

![requests-ip](../assets/requests-ip.png)

É IDEAL que poucas coisas sejam acessíveis à internet, e que a maior parte da infraestrutura seja interna, para diminuir a superfície de ataque e a complexidade da infraestrutura.

---

### NAT Gateway
- O NAT Gateway é um serviço que permite que os recursos internos se comuniquem com a internet, mas que não permite que a internet se comunique com os recursos internos.
- Ele é usado para permitir que os recursos internos acessem a internet, mas que não permita que a internet acesse os recursos internos.
- Ele é configurado em uma sub-rede pública e é responsável por traduzir os endereços IP internos para endereços IP públicos.

---

### DNS Routing

1. Geolocation
   - O DNS Routing é usado para redirecionar o tráfego de acordo com a localização do usuário.
2. Failover
   - O Failover é usado para redirecionar o tráfego para outro servidor em caso de falha.
3. Latency-Based
   - O Latency-Based é usado para redirecionar o tráfego para o servidor com menor latência.
4. IP-Based
   - O IP-Based é usado para redirecionar o tráfego para o servidor com o endereço IP especificado.
5. Weighted
   - O Weighted é usado para redirecionar o tráfego para o servidor com base em um peso especificado.
6. Multi-Awnser
   - O Multi-Awnser é usado para redirecionar o tráfego para vários servidores em caso de falha.
7. Round-Robin
   - O Round-Robin é usado para distribuir o tráfego entre vários servidores em ordem circular.
