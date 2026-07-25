# Event Sourcing

Event Sourcing é um padrão em que a fonte de verdade do sistema é uma sequência append-only de eventos de domínio. O estado atual não é salvo como a única verdade: ele é obtido aplicando os eventos em ordem.

Um evento de domínio representa um fato significativo, como `AccountOpened`, `MoneyDeposited` ou `MoneyWithdrawn`. Registrar apenas alterações genéricas, como `AccountUpdated`, pode não ser suficiente para reconstruir o comportamento e as regras do domínio.

## Exemplo: conta bancária

Uma conta pode ter os seguintes eventos:

```json
[
  {
    "sequence": 1,
    "type": "AccountOpened",
    "data": {
      "account_id": "acc-001"
    }
  },
  {
    "sequence": 2,
    "type": "MoneyDeposited",
    "data": {
      "amount": 100.0,
      "currency": "BRL"
    }
  },
  {
    "sequence": 3,
    "type": "MoneyWithdrawn",
    "data": {
      "amount": 30.0,
      "currency": "BRL"
    }
  }
]
```

Ao reproduzir a sequência, o sistema chega ao saldo de `70.0 BRL`. A regra que valida um saque pode usar o estado obtido até aquele momento; um saque não deve ser aceito se a conta ainda não tiver saldo suficiente.

A ordenação precisa ser definida por agregado, como uma conta ou um pedido. Um timestamp ajuda na auditoria, mas não deve ser usado sozinho como garantia de ordenação em um sistema distribuído. É comum usar um número de sequência e uma versão esperada para detectar atualizações concorrentes.

## Exemplo: pedido

Um fluxo de pedido poderia ser representado por eventos como:

1. `OrderCreated` — pedido criado.
2. `ItemAdded` — item adicionado.
3. `ItemRemoved` — item removido.
4. `PaymentApproved` — pagamento aprovado, possivelmente após um webhook externo.
5. `OrderShipped` — pedido enviado.

Um carrinho pode ser um estado mais mutável e temporário, enquanto um pedido costuma ter um histórico de transições mais relevante para o negócio. A escolha por Event Sourcing deve vir da necessidade desse histórico e da reconstrução do estado, não apenas do fato de existirem eventos.

## Propriedades dos eventos

- **Imutabilidade:** um evento já registrado não deve ser editado para mudar o passado.
- **Significado de negócio:** o evento explica o que ocorreu, e não apenas qual coluna foi alterada.
- **Ordenação:** normalmente existe uma ordem por agregado; não é necessário haver uma ordem global de todos os eventos do sistema.
- **Dados suficientes:** o payload precisa permitir a reprodução do estado. Alterar o schema ao longo do tempo exige versionamento, upcasting ou outra estratégia de compatibilidade.

Se uma operação precisar ser corrigida, o caminho normal é publicar um novo evento de compensação ou correção. Alterar eventos antigos destrói a trilha histórica e pode produzir estados diferentes para consumidores que já fizeram a reprodução.

## Projeções e snapshots

Reproduzir milhares ou milhões de eventos a cada consulta pode ser caro. Por isso, o sistema pode:

- manter projeções ou modelos de leitura atualizados a partir dos eventos;
- criar snapshots periódicos do estado e reproduzir apenas os eventos posteriores ao snapshot;
- reprocessar os eventos quando for necessário criar uma nova projeção.

As projeções podem ficar temporariamente atrasadas, e precisam de idempotência, retries e monitoramento. Se uma projeção for perdida, ela só poderá ser recriada se os eventos continuarem disponíveis e forem compatíveis com o código atual.

## Vantagens

- **Histórico completo:** facilita auditoria, investigação e visualização do estado em diferentes momentos.
- **Replay:** permite reconstruir o estado e criar novas projeções a partir dos eventos existentes.
- **Regras temporais:** o domínio pode reagir a uma sequência de fatos, e não apenas ao valor atual de uma tabela.
- **Integração:** os eventos podem alimentar outros modelos e serviços, desde que sejam publicados e consumidos com contratos claros.

## Trade-offs

- **Complexidade de domínio:** é preciso definir bons eventos e manter a compatibilidade deles ao longo do tempo.
- **Custo de armazenamento e operação:** o log cresce e exige retenção, backup e monitoramento.
- **Consistência eventual:** projeções e consumidores podem não estar atualizados imediatamente.
- **Privacidade e correções:** remover ou alterar dados históricos pode ser difícil quando há requisitos legais ou dados pessoais.
- **Debugging:** um erro no replay ou na evolução do schema pode afetar todas as projeções.

Event Sourcing não é o mesmo que salvar um log de auditoria ao lado de um CRUD. Se o banco de tabelas continua sendo a fonte de verdade e o log é apenas informativo, isso é um audit log, não Event Sourcing.

Também não é sinônimo de Event-Driven ou CQRS. Event Sourcing define como o estado é persistido; Event-Driven define uma forma de comunicação; CQRS separa modelos de leitura e escrita. Os três padrões podem ser combinados, mas são decisões independentes.

Para uma entidade simples, sem necessidade de histórico ou replay, um CRUD tradicional costuma ser mais fácil de manter. Em uma entrevista, vale propor Event Sourcing quando auditoria, histórico temporal, reconstrução do estado ou múltiplas projeções forem requisitos reais.
