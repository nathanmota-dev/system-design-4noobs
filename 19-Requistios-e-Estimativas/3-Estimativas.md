# Estimativas para System Design

Em entrevistas de System Design, estimativas rápidas, também chamadas de *back-of-the-envelope calculations*, servem para verificar se uma arquitetura consegue atender aos requisitos antes de começar a desenhar seus componentes.

A maioria das entrevistas não exige precisão matemática. O objetivo é entender a ordem de grandeza do sistema para responder perguntas como:

* Quanto armazenamento será necessário?
* Quantas requisições o sistema receberá?
* Qual será a largura de banda utilizada?
* O volume de dados quentes cabe em memória ou em uma camada de cache?
* Um Object Storage é suficiente para atender à latência exigida?
* Será necessário utilizar cache, CDN ou armazenamento em SSD/NVMe?

> O mais importante não é chegar a um número exato, mas deixar claras as premissas utilizadas e chegar a uma estimativa razoável.

---

## 1. Ordens de Grandeza de Latência

Os valores abaixo são aproximações utilizadas para comparar o custo de diferentes operações. Eles podem variar conforme o hardware, a infraestrutura, a tecnologia utilizada e a distância entre os servidores.

| Operação                              | Tempo aproximado |
| :------------------------------------ | :--------------- |
| Acesso à memória RAM                  | ~100 ns          |
| Comprimir 1 KB                        | ~10 µs           |
| Transferir 2 KB em uma rede de 1 Gbps | ~20 µs           |
| Round trip dentro de um datacenter    | ~500 µs          |
| Busca em um disco rígido — HDD        | ~10 ms           |
| Transferir 1 MB em uma rede de 1 Gbps | ~8–10 ms         |
| Round trip entre regiões distantes    | ~100–200 ms      |

Esses números não devem ser tratados como valores fixos. O objetivo é compreender a diferença entre as ordens de grandeza.

Como referência aproximada, um piscar de olhos dura algumas centenas de milissegundos. Isso ajuda a visualizar a ordem de grandeza, mas não significa que toda latência abaixo de 100 ms seja imperceptível: o usuário percebe o tempo total da interação, que pode reunir várias operações.

Por exemplo:

* Nanossegundos (`ns`) são menores que microssegundos.
* Microssegundos (`µs`) são menores que milissegundos.
* Acessos à memória normalmente são mais rápidos que acessos a disco.
* Chamadas de rede adicionam latência ao sistema.
* Quanto maior a distância geográfica, maior tende a ser a latência.

> **Principal conclusão:** o requisito de latência pode influenciar diretamente onde os dados serão mantidos. Mesmo que um Object Storage seja adequado para armazenamento, ele pode não ser suficiente sozinho quando o sistema precisa entregar conteúdo com uma latência muito baixa.

---

## 2. Exemplo Prático: Armazenamento de Stories

Imagine um sistema de stories semelhante ao Instagram.

Nesse tipo de aplicação, o conteúdo possui dois momentos diferentes:

1. Durante as primeiras 24 horas, o story pode ser visualizado por outros usuários e tende a receber a maior parte dos acessos.
2. Depois de 24 horas, o story deixa de ficar publicamente disponível e é movido para o arquivo pessoal do usuário.

Isso cria dois grupos de dados:

* **Dados quentes:** stories publicados nas últimas 24 horas.
* **Dados frios:** stories antigos armazenados no arquivo pessoal.

Essa separação é importante porque os dois grupos possuem requisitos diferentes de acesso, latência e custo.

---

## 3. Premissas do Cenário

Para simplificar, vamos considerar:

* **Usuários ativos diariamente:** 500.000
* **Publicações:** 1 story por usuário por dia
* **Tamanho original da imagem:** 2 MB
* **Tamanho após compressão:** 500 KB, ou aproximadamente 0,5 MB
* **Tempo em que o story permanece ativo:** 24 horas

Também vamos considerar que todos os usuários ativos publicam um story por dia.

---

## 4. Estimativa do Volume Diário

O volume diário seria:

**500.000 stories/dia × 0,5 MB = 250.000 MB/dia**

Convertendo para gigabytes:

**250.000 MB ≈ 250 GB/dia**

Portanto, o conjunto de stories ativos nas últimas 24 horas teria aproximadamente:

**250 GB**

Esse valor representa uma aproximação do **working set**, ou seja, o conjunto de dados que está sendo utilizado com maior frequência pelo sistema.

---

## 5. Armazenamento ao Longo do Tempo

Embora apenas 250 GB estejam ativos por vez, os stories podem continuar armazenados no arquivo pessoal dos usuários.

Sem considerar exclusões, o volume acumulado seria aproximadamente:

| Período | Armazenamento acumulado |
| :------ | ----------------------: |
| 1 dia   |                  250 GB |
| 30 dias |                  7,5 TB |
| 1 ano   |                91,25 TB |

Essa estimativa ainda não considera:

* Diferentes resoluções da imagem;
* Miniaturas;
* Vídeos;
* Metadados;
* Replicação;
* Backups;
* Crescimento da quantidade de usuários;
* Overhead de armazenamento;
* Stories excluídos pelos usuários.

Na prática, o espaço necessário seria maior.

---

## 6. Estimativa de Uploads por Segundo

Um dia possui:

**24 × 60 × 60 = 86.400 segundos**

Portanto:

**500.000 uploads / 86.400 segundos ≈ 5,8 uploads por segundo**

Essa é apenas a média.

Os usuários não publicam stories de maneira uniforme durante todo o dia. Em determinados horários, a quantidade de uploads pode ser muito maior.

Se considerarmos um pico dez vezes maior que a média:

**5,8 × 10 ≈ 58 uploads por segundo**

O sistema poderia ser projetado inicialmente para receber aproximadamente **60 uploads por segundo durante períodos de pico**.

Porém, a quantidade de leituras tende a ser muito maior que a quantidade de uploads, já que cada story pode ser visualizado por várias pessoas.

Por isso, seria necessário estimar separadamente:

* Uploads por segundo;
* Visualizações por segundo;
* Largura de banda de entrada;
* Largura de banda de saída;
* Taxa de acerto do cache;
* Quantidade média de visualizações por story.

---

## 7. Impacto da Latência na Arquitetura

Em uma implementação simples, os stories poderiam ser armazenados diretamente em um Object Storage, como Amazon S3, Cloudflare R2 ou um serviço equivalente, e distribuídos por uma CDN.

```text
Cliente
  ↓
CDN
  ↓
Object Storage
```

Essa arquitetura pode ser suficiente para muitos sistemas.

Porém, em uma entrevista, o requisito pode exigir uma latência muito baixa e previsível para acessar os stories ativos.

Nesse caso, é necessário avaliar se acessar o Object Storage em cada cache miss atenderia ao SLA definido.

Como o conjunto quente estimado possui aproximadamente 250 GB, pode fazer sentido manter os stories ativos em uma camada dedicada de armazenamento rápido.

```text
Cliente
  ↓
CDN
  ↓
Cache ou Hot Storage
  ↓
Object Storage
```

Essa camada poderia utilizar:

* Memória RAM;
* Redis ou outro cache distribuído;
* SSD ou NVMe local;
* Servidores próprios de cache de mídia;
* Uma combinação de memória e disco rápido.

A escolha depende principalmente da latência exigida.

### Exemplo

Se o requisito for servir stories em poucos milissegundos, uma camada de cache dedicada pode ser necessária.

Se uma latência um pouco maior for aceitável, uma CDN combinada com Object Storage pode ser suficiente.

Portanto, não é o tipo de arquivo que determina sozinho onde ele deve ser armazenado. A decisão depende de fatores como:

* SLA de latência;
* Volume do conjunto quente;
* Frequência de acesso;
* Taxa esperada de cache hit;
* Custo da memória;
* Custo de transferência;
* Quantidade de réplicas;
* Tolerância a falhas.

---

## 8. Ciclo de Vida dos Stories

Como os stories ficam disponíveis publicamente por apenas 24 horas, é possível criar uma política de armazenamento baseada em temperatura.

### Durante as primeiras 24 horas

O story permanece no **Hot Storage**, pois possui maior probabilidade de acesso.

```text
Upload
  ↓
Compressão e processamento
  ↓
Hot Storage ou cache dedicado
  ↓
CDN
```

O Object Storage ainda pode ser utilizado como fonte persistente e durável.

Uma possível arquitetura seria:

```text
Cliente
  ↓
API de upload
  ↓
Object Storage
  ↓
Processamento assíncrono
  ↓
Cache ou Hot Storage
  ↓
CDN
```

Nesse cenário:

* O Object Storage mantém a cópia durável do arquivo.
* O cache ou Hot Storage atende às leituras com baixa latência.
* A CDN aproxima o conteúdo dos usuários.
* O processamento assíncrono gera versões comprimidas e diferentes resoluções.

### Depois de 24 horas

O story deixa de ficar disponível publicamente e passa a ser acessado apenas pelo proprietário por meio do arquivo pessoal.

Como a frequência de acesso tende a diminuir, ele pode ser removido da camada de cache e continuar apenas no Object Storage.

```text
Hot Storage
  ↓
Expiração após 24 horas
  ↓
Object Storage
```

Posteriormente, dependendo da frequência de acesso e dos requisitos do produto, o arquivo também pode ser movido para uma classe de armazenamento mais barata.

```text
Object Storage padrão
  ↓
Armazenamento de acesso infrequente
  ↓
Cold Storage ou Archive Storage
```

Essa movimentação pode ser feita por meio de políticas automáticas de ciclo de vida.

---

## 9. Cache não é a Fonte de Verdade

Mesmo que todos os stories ativos sejam mantidos em cache, o cache não deveria ser a única cópia do conteúdo.

O arquivo original ou processado deve continuar armazenado em uma camada durável.

```text
Cache
  └── cópia temporária para acesso rápido

Object Storage
  └── fonte persistente e durável
```

Isso é importante porque a camada de cache pode sofrer:

* Eviction;
* Reinicializações;
* Falhas de nós;
* Perda de dados;
* Redistribuição do cluster;
* Expiração automática.

Caso ocorra um cache miss, o sistema pode buscar o objeto no armazenamento persistente, devolvê-lo ao usuário e inseri-lo novamente no cache.

```text
Requisição
  ↓
Busca no cache
  ├── Cache hit → retorna o story
  └── Cache miss → busca no Object Storage
                         ↓
                    atualiza o cache
                         ↓
                    retorna o story
```

---

## 10. O Conjunto Quente Pode Ser Menor

Os 250 GB representam todos os stories publicados em um período de 24 horas.

Porém, nem todos eles necessariamente precisam permanecer em RAM.

É possível que apenas uma parte dos stories concentre a maior parte dos acessos.

Por exemplo:

* Stories de usuários com muitos seguidores;
* Stories recém-publicados;
* Stories visualizados pelo usuário atual;
* Conteúdo popular em determinada região;
* Próximos stories de uma sequência já iniciada.

Nesse caso, o sistema poderia manter:

* Todos os stories ativos em SSD/NVMe;
* Apenas os stories mais acessados em RAM;
* Todo o conteúdo persistido no Object Storage;
* Cópias distribuídas na CDN.

```text
RAM
  └── conteúdo extremamente quente

SSD/NVMe
  └── conjunto de stories ativos

Object Storage
  └── cópia durável de todos os stories

Cold Storage
  └── stories antigos e pouco acessados
```

Essa abordagem cria diferentes camadas de armazenamento com custos e latências diferentes.

---

## 11. Conclusão do Design

A estimativa mostra que o sistema receberia aproximadamente **250 GB de novos stories por dia**.

Como os stories ficam disponíveis publicamente durante apenas 24 horas, o conjunto quente seria relativamente previsível e teria aproximadamente o mesmo tamanho do volume diário.

Dependendo do requisito de latência, pode fazer sentido manter esse conjunto em uma camada dedicada de cache ou Hot Storage.

Uma possível arquitetura seria composta por:

* **Object Storage** como fonte durável dos arquivos;
* **CDN** para distribuir o conteúdo geograficamente;
* **Cache em RAM** para stories extremamente acessados;
* **SSD ou NVMe** para manter um conjunto quente maior com menor custo;
* **Banco de dados** para armazenar metadados e controlar a expiração;
* **Processamento assíncrono** para compressão e geração de resoluções;
* **Políticas de ciclo de vida** para mover conteúdos antigos para classes mais baratas.

Após 24 horas, o story pode ser:

1. Removido da CDN e da camada de cache;
2. Marcado como não disponível publicamente;
3. Mantido no Object Storage para acesso pelo proprietário;
4. Movido posteriormente para uma classe de armazenamento mais barata.

> **Principal conclusão:** Object Storage e CDN seriam o padrão para persistência e distribuição, mas requisitos muito baixos de latência podem justificar uma camada dedicada de cache ou Hot Storage. Como os stories possuem uma janela de alta utilização de apenas 24 horas, o sistema consegue estimar com maior facilidade o tamanho do conjunto quente e definir quando o conteúdo deve ser removido das camadas mais rápidas e caras.
