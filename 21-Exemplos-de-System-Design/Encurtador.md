# Encurtador de URL

**1 - Requisitos Funcionais**

* Receber uma URL original e gerar uma URL curta.
* Redirecionar o usuário da URL curta para a URL original.
* Reutilizar a mesma URL curta quando a URL original for idêntica.
* Não exigir login nem rastrear cliques neste escopo.

**2 - Requisitos Não Funcionais**

* Alta disponibilidade: um link existente precisa continuar funcionando.
* Baixa latência no redirecionamento, principalmente no caminho de leitura.
* Suportar aproximadamente 50 milhões de requisições por mês.
* Durabilidade dos mapeamentos e recuperação após falhas.
* Arquitetura predominantemente orientada a leitura e capaz de crescer horizontalmente.
* Proteção contra abuso, URLs maliciosas e excesso de requisições.

**3 - Pontos de Atenção**

* Concorrência na criação da mesma URL e garantia de idempotência.
* Colisão e tamanho do código curto.
* Disponibilidade do banco e comportamento durante falhas.
* Cache de redirecionamentos e invalidação em caso de alteração ou remoção.
* Crescimento do banco, índices, backup e restauração.
* Códigos sequenciais são fáceis de enumerar; se isso for um risco, usar códigos aleatórios.
* Validação e rate limiting para reduzir phishing, spam e abuso da API.

**4 - Estimativas (BoE)**

* 50 milhões de requisições por mês.
* RPS médio: `50.000.000 / (30 × 24 × 3.600) ≈ 19,3 req/s`.
* Considerando um pico de 10 vezes a média: aproximadamente 200 req/s.
* Se 80% do volume mensal representar URLs novas e únicas: `50 milhões × 0,8 = 40 milhões` de novos links por mês.
* Em 3 anos: `40 milhões × 36 = 1,44 bilhão` de links.
* Base 62 (`A-Z`, `a-z`, `0-9`):
  * `62^5 = 916.132.832`, insuficiente para 1,44 bilhão.
  * `62^6 = 56.800.235.584`, suficiente para a estimativa.
* O armazenamento deve considerar o tamanho médio da URL, metadados, índices, replicação e retenção.

**5 - Overview de Alto Nível**

```text
[Cliente]
    |
[CDN opcional / API Gateway / Load Balancer]
    |
[URL Service - stateless]
    | \
    |  \--> [Redis: short_code -> original_url]
    \-----> [Banco SQL com alta disponibilidade]
```

O serviço valida a criação, gera o código e persiste o mapeamento. No redirecionamento, consulta primeiro o cache e usa o banco como fonte de verdade em caso de *cache miss*.

**6 - Schema DB**

```text
urls
----
id            BIGINT       PRIMARY KEY
short_code    VARCHAR(6)   UNIQUE NOT NULL
original_url  TEXT         UNIQUE NOT NULL
created_at    TIMESTAMP    NOT NULL
```

* `id` é usado para gerar o `short_code` por conversão de base 10 para base 62.
* A restrição única em `original_url` permite reutilizar o código de uma URL já cadastrada.
* O índice único em `short_code` garante que cada código aponte para um único destino.
* O banco relacional é a fonte de verdade; Redis e CDN são camadas de aceleração.

**7 - Design da API**

Criar ou reutilizar uma URL curta:

```http
POST /v1/shorter
Content-Type: application/json

{
  "original_url": "https://www.exemplo.com/pagina"
}
```

```json
{
  "original_url": "https://www.exemplo.com/pagina",
  "short_url": "https://leolinkcurto.com/w7E"
}
```

Retornar `201 Created` para uma nova URL e `200 OK` quando o mapeamento já existir. URLs inválidas retornam `400 Bad Request`; excesso de requisições retorna `429 Too Many Requests`.

Redirecionar:

```http
GET /w7E
```

Resposta para um código válido: `301 Moved Permanently` com `Location: https://www.exemplo.com/pagina`. Um código inexistente retorna `404 Not Found`. Se o destino puder mudar ou expirar, `302` ou `307` evita que clientes armazenem um redirecionamento permanente.

**8 - Flow**

Criação da URL:

1. O cliente envia a URL original.
2. O serviço valida o formato e aplica as regras de segurança.
3. O serviço consulta `original_url`.
4. Se já existir, devolve o `short_code` existente.
5. Caso contrário, reserva um `id`, converte-o para base 62 e grava o registro em uma transação.
6. Em uma disputa concorrente, a restrição única em `original_url` resolve a corrida; a requisição pode reler o registro vencedor.
7. O serviço retorna a URL curta e pode aquecer o cache.

Redirecionamento:

1. O cliente solicita `GET /{short_code}`.
2. CDN ou Redis procura o código.
3. Em *cache hit*, o serviço responde com `301` e o header `Location`.
4. Em *cache miss*, o serviço consulta o banco, grava o resultado no cache e responde.
5. Se o código não existir, responde `404`.
6. Ao alterar ou remover um link, o valor deve ser invalidado no Redis e na CDN.

**9 - SD Completo**

```text
                         +-----------+
                         |  Cliente  |
                         +-----+-----+
                               |
                    +----------v-----------+
                    | CDN opcional / Edge  |
                    +----------+-----------+
                               |
                    +----------v-----------+
                    | Gateway / Load Bal.  |
                    +----------+-----------+
                               |
                    +----------v-----------+
                    | URL Service (N)      |
                    +------+-----------+---+
                           |           |
                +----------v--+   +----v----------------+
                | Redis       |   | Banco SQL HA         |
                | cache       |   | primary + réplicas   |
                +-------------+   +----------------------+
```

* O `URL Service` é stateless e pode ser replicado horizontalmente.
* O banco mantém as restrições de unicidade e recebe backups e replicação.
* Redis e CDN reduzem a latência e a carga no banco para redirecionamentos frequentes.
* Timeouts, health checks, rate limiting e circuit breaker protegem as dependências.
* A conversão de ID para base 62 evita colisões, mas revela uma sequência previsível. Se a enumeração for inaceitável, troque por um código aleatório com índice único e retry em colisões.
