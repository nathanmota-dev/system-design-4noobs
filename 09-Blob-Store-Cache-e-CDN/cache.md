# Cache

Cache é um mecanismo de armazenamento temporário utilizado para reduzir o tempo de acesso a dados e melhorar o desempenho de um sistema.

Em vez de buscar ou processar uma informação novamente em sua fonte original, como um banco de dados, uma API ou uma operação computacional custosa, o sistema pode armazenar temporariamente o resultado e reutilizá-lo nas próximas requisições.

O cache geralmente utiliza um armazenamento mais rápido do que a fonte original. Em muitos casos, ele utiliza memória RAM, que possui algumas características:

* É mais rápida.
* Possui menor capacidade de armazenamento.
* É mais cara por unidade de armazenamento.
* É volátil, podendo perder os dados quando o serviço é reiniciado.

Por causa dessas limitações, não é recomendado armazenar todos os dados em cache. Normalmente, são bons candidatos:

* Dados acessados com frequência.
* Resultados de consultas custosas ao banco de dados.
* Resultados de cálculos ou processamentos demorados.
* Respostas de APIs externas.
* Arquivos estáticos, como imagens, CSS e JavaScript.
* Dados que podem permanecer desatualizados por um pequeno período.

## Exemplo

Imagine o perfil do Cristiano Ronaldo no Instagram.

Por ser uma conta com muitos seguidores, o perfil pode receber um grande número de acessos. Caso cada requisição precise consultar diretamente o banco de dados, o sistema realizará diversas consultas repetidas para obter praticamente as mesmas informações.

Uma alternativa seria armazenar temporariamente em cache os dados mais acessados do perfil. Assim, diversas requisições poderiam ser respondidas utilizando o cache, reduzindo a quantidade de consultas realizadas no banco de dados.

Entretanto, informações que mudam com frequência, como o número de seguidores, precisam de uma estratégia para atualizar ou invalidar o cache.

## Objetivos do cache

O uso de cache pode ajudar a:

* Reduzir consultas e operações custosas.
* Diminuir a latência das respostas.
* Reduzir a carga sobre bancos de dados e serviços externos.
* Aumentar a quantidade de usuários que o sistema consegue atender.
* Melhorar a escalabilidade e a responsividade da aplicação.

## Cache hit e cache miss

Quando o sistema procura um dado no cache, podem ocorrer duas situações.

### Cache hit

Acontece quando o dado solicitado já está disponível no cache.

Nesse caso, o sistema consegue retornar a resposta rapidamente, sem consultar a fonte original.

### Cache miss

Acontece quando o dado não está presente no cache ou já expirou.

Nesse caso, o sistema precisa buscar o dado na fonte original. Depois disso, pode armazená-lo no cache para atender mais rapidamente às próximas requisições.

Um fluxo comum seria:

1. A aplicação procura o dado no cache.
2. Caso encontre, retorna o dado.
3. Caso não encontre, consulta o banco de dados.
4. Armazena o resultado no cache.
5. Retorna o dado para o cliente.

## Expiração do cache

Os dados armazenados em cache normalmente possuem um tempo de validade, conhecido como TTL — Time to Live.

Por exemplo, um dado armazenado com TTL de cinco minutos será removido ou considerado expirado após esse período.

O TTL ajuda a evitar que informações antigas permaneçam indefinidamente no cache.

A escolha do tempo de expiração depende do tipo de dado:

* Dados que mudam frequentemente devem ter um TTL menor.
* Dados que raramente mudam podem ter um TTL maior.
* Dados críticos podem exigir invalidação imediata quando forem alterados.

## Invalidação do cache

Invalidar o cache significa remover ou atualizar um dado armazenado quando sua fonte original é modificada.

Esse é um dos principais desafios relacionados ao uso de cache. Caso o banco de dados seja atualizado, mas o cache continue com o valor antigo, os usuários podem receber informações desatualizadas.

Uma estratégia comum é remover o item do cache após uma operação de escrita. Na próxima leitura, acontecerá um cache miss e o cache será preenchido novamente com o valor atualizado.

## Políticas de remoção do cache

Como o cache possui espaço limitado, é necessário definir quais dados serão removidos quando sua capacidade for atingida. Essas regras são chamadas de políticas de remoção, ou eviction policies.

Uma das estratégias mais utilizadas é a LRU — Least Recently Used.

### LRU — Least Recently Used

A LRU remove o item que não é utilizado há mais tempo.

A ideia é assumir que dados acessados recentemente possuem maior chance de serem acessados novamente, enquanto dados que não são utilizados há muito tempo podem ser removidos.

Exemplo:

Imagine que o cache possui espaço para três itens:

1. Perfil A
2. Perfil B
3. Perfil C

Se o Perfil A for acessado novamente, ele passa a ser considerado o mais recentemente utilizado.

Caso seja necessário inserir o Perfil D, o cache removerá o item que está há mais tempo sem ser acessado, que nesse exemplo poderia ser o Perfil B.

A ordem ficaria aproximadamente assim:

```text
Antes do acesso:
A → B → C

Após acessar A:
B → C → A

Após inserir D:
C → A → D
```

Nesse exemplo, o Perfil B foi removido por ser o menos recentemente utilizado.

Outras políticas comuns incluem:

* FIFO — remove o item inserido há mais tempo.
* LFU — remove o item acessado com menor frequência.
* TTL — remove o item quando seu tempo de validade expira.
* Random — remove um item aleatoriamente.

A escolha da política depende do padrão de acesso aos dados. A LRU costuma ser uma boa estratégia padrão para cenários em que itens acessados recentemente tendem a ser reutilizados.

## Cache no servidor

O cache pode ser implementado dentro da própria aplicação ou em um serviço separado.

Uma aplicação pode utilizar um cache local, armazenado na memória do próprio processo. Essa opção é simples e rápida, mas cada instância da aplicação terá seu próprio cache.

Em sistemas distribuídos, é comum utilizar um cache externo e compartilhado, como:

* Redis.
* Memcached.

Nesse cenário, diferentes instâncias da aplicação conseguem acessar o mesmo cache.

Também existem serviços gerenciados de cache, como:

* Amazon ElastiCache.
* Azure Cache for Redis.
* Google Cloud Memorystore.

O uso de um serviço externo facilita o compartilhamento do cache entre várias instâncias, mas adiciona uma comunicação pela rede.

## Cache para arquivos estáticos

Arquivos estáticos podem ser armazenados em cache no navegador, em proxies reversos ou em uma CDN.

Uma CDN — Content Delivery Network — distribui cópias de conteúdos por servidores localizados em diferentes regiões geográficas.

Quando um usuário solicita um arquivo, a CDN pode entregá-lo por meio de um servidor mais próximo, reduzindo a latência e diminuindo a carga sobre o servidor de origem.

Alguns conteúdos normalmente distribuídos por uma CDN são:

* Imagens.
* Vídeos.
* Arquivos CSS.
* Arquivos JavaScript.
* Fontes.
* Documentos.
* Páginas estáticas.

A CDN funciona como uma camada de cache distribuída, mas também pode oferecer outros recursos, como proteção contra ataques, compressão, certificados TLS e balanceamento de tráfego.

## Cache no cliente

O cache no cliente ocorre no dispositivo do usuário, normalmente no navegador.

O navegador pode armazenar recursos como:

* Imagens.
* Arquivos CSS.
* Arquivos JavaScript.
* Fontes.
* Respostas HTTP.
* Dados da aplicação.

O comportamento do cache HTTP pode ser controlado por cabeçalhos como:

* `Cache-Control`.
* `Expires`.
* `ETag`.
* `Last-Modified`.

Também é possível armazenar dados utilizando recursos como:

* Local Storage.
* Session Storage.
* IndexedDB.
* Cache API.

Entretanto, essas tecnologias possuem propósitos diferentes. Local Storage e Session Storage, por exemplo, são mecanismos de armazenamento no navegador, mas não substituem diretamente o cache HTTP.

## Trade-offs do cache

Apesar de melhorar o desempenho, o cache adiciona complexidade ao sistema.

Alguns de seus principais trade-offs são:

* Possibilidade de servir dados desatualizados.
* Maior complexidade para invalidar dados.
* Consumo adicional de memória ou armazenamento.
* Necessidade de definir corretamente o TTL.
* Possibilidade de muitos cache misses simultâneos.
* Necessidade de lidar com indisponibilidade do serviço de cache.

Por isso, cache não deve ser utilizado apenas porque um dado pode ser armazenado. É necessário avaliar se o ganho de desempenho compensa a complexidade adicionada.


## Problemas com o cache

Stampede - O problema de stampede ocorre quando múltiplos clientes acessam o cache simultaneamente, resultando em múltiplas consultas ao banco de dados.
Hot keys - O problema de hot keys ocorre quando um único chave é acessada muito frequentemente, resultando em um grande número de consultas ao banco de dados.