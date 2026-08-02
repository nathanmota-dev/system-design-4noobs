## Resumo

As principais estratégias de otimização de banco de dados são:

* usar cache para reduzir carga no banco;
* criar índices com base nos padrões de acesso;
* usar connection pooling para reaproveitar conexões;
* usar read replicas quando há muito mais leitura do que escrita;
* usar partitioning para dividir tabelas muito grandes;
* usar sharding para escalar horizontalmente;
* entender os trade-offs do Teorema CAP;
* normalizar para consistência;
* denormalizar para leitura rápida;
* otimizar queries com EXPLAIN, paginação, batching e materialized views.

Em entrevistas de System Design, o mais importante não é apenas citar a técnica, mas explicar:

```txt
Qual problema ela resolve?
Qual trade-off ela cria?
Em qual cenário ela faz sentido?
```
