## 6. Teorema CAP

O Teorema CAP é um conceito importante em sistemas distribuídos.

Ele afirma que, em um sistema distribuído, diante de uma falha de partição de rede, não é possível garantir simultaneamente:

* **Consistency**;
* **Availability**;
* **Partition Tolerance**.

### Consistency

Todos os nós enxergam o mesmo dado ao mesmo tempo.

Exemplo:

```txt
Se uma transferência bancária foi confirmada, qualquer leitura posterior deve refletir esse novo saldo.
```

### Availability

O sistema continua respondendo, mesmo que algumas partes estejam com problema.

Exemplo:

```txt
Mesmo se um nó falhar, a aplicação ainda responde às requisições.
```

### Partition Tolerance

O sistema continua funcionando mesmo quando existe falha de comunicação entre nós.

Exemplo:

```txt
Dois servidores não conseguem se comunicar temporariamente, mas o sistema precisa lidar com isso.
```

### Interpretação prática

Em sistemas distribuídos, falhas de rede podem acontecer. Por isso, normalmente assumimos que **Partition Tolerance** é necessária.

Quando ocorre uma partição, o sistema precisa escolher entre:

* priorizar consistência;
* priorizar disponibilidade.

Exemplo com prioridade em consistência:

```txt
Se não for possível garantir o dado correto, o sistema pode recusar a operação.
```

Exemplo com prioridade em disponibilidade:

```txt
O sistema continua respondendo, mesmo que alguns dados possam estar temporariamente desatualizados.
```

Em uma entrevista, o ponto principal é explicar o trade-off.

Exemplos:

* sistema bancário tende a priorizar consistência;
* feed de rede social pode aceitar consistência eventual;
* carrinho de compras pode aceitar alguma inconsistência temporária;
* estoque e pagamento geralmente exigem mais consistência.

---

