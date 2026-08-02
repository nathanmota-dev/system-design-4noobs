## 7. Normalização e Denormalização

A modelagem dos dados também afeta performance.

Duas estratégias importantes são:

* normalização;
* denormalização.

---

### 7.1. Normalização

Normalização é o processo de dividir os dados em tabelas menores e relacionadas para reduzir redundância e evitar inconsistências.

Exemplo ruim:

```txt
Tabela users:
- id
- name
- email
- bank_account_number
- bank_name
- bank_branch
```

Podemos separar em:

```txt
Tabela users:
- id
- name
- email

Tabela bank_accounts:
- id
- user_id
- account_number
- bank_name
- bank_branch
```

Vantagens:

* reduz duplicação;
* melhora integridade;
* facilita atualização de dados;
* evita inconsistência.

Desvantagens:

* pode exigir joins;
* algumas leituras ficam mais caras;
* queries podem ficar mais complexas.

Normalização costuma ser uma boa escolha quando consistência e integridade dos dados são mais importantes.

---

### 7.2. Denormalização

Denormalização é duplicar dados de propósito para melhorar a performance de leitura.

Exemplo:

```txt
Tabela orders:
- id
- user_id
- user_name
- user_email
- total
```

Mesmo que `user_name` e `user_email` já existam na tabela de usuários, eles podem ser duplicados em `orders` para evitar joins frequentes.

Vantagens:

* leituras mais rápidas;
* menos joins;
* queries mais simples;
* útil para dados muito acessados.

Desvantagens:

* maior uso de armazenamento;
* risco de inconsistência;
* atualizações ficam mais difíceis;
* pode exigir eventos ou jobs para manter dados sincronizados.

Denormalização é comum em sistemas de alta escala, principalmente quando a aplicação tem padrões de leitura muito previsíveis.

Exemplos:

* feed de rede social;
* catálogo de produtos;
* dashboards;
* relatórios;
* sistemas com muitos reads e poucos writes.

---
