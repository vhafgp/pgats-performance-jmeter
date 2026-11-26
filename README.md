# Testes de performance da API de Pets com JMeter

Trabalho de JMeter da disciplina de Automação de Testes de Performance da PGATS. O plano testa a [API de Pets](https://github.com/tiagotaq/api-de-pets) com um thread group de modelo fechado e um de modelo aberto. Os dois fazem login e depois chamam `POST /pets` e `GET /pets`.

## Estrutura

```text
apiDePets.jmx        plano de teste
fixtures/pets.csv    nomes e espécies dos pets
```

## O plano

- HTTP Request Defaults aponta para `localhost:3000` e um HTTP Header Manager manda `Content-Type: application/json` em todas as requisições.
- O setUp Thread Group faz o login uma vez com o usuário `tiago`. O JSON Extractor pega o `$.token` e um JSR223 PostProcessor guarda o token na propriedade `tokenProp`, que os outros grupos leem com `${__property(tokenProp)}`.
- Modelo fechado: Thread Group com 50 usuários, ramp-up de 10 s e 5 iterações.
- Modelo aberto: Open Model Thread Group com `rate(0) random_arrivals(10 sec) rate(20/sec) random_arrivals(40 sec) random_arrivals(10 sec) rate(0)`. Sobe até 20 req/s em 10 s, fica 40 s nisso e desce em 10 s.
- Os dois grupos leem o `fixtures/pets.csv` e mandam `Authorization: Bearer` com o token. O nome do pet leva `${__time()}` e `${__counter(false)}` porque a API não aceita nome repetido.
- Asserção no status code: 200 no login e no `GET /pets`, 201 no `POST /pets`.
- Listeners: View Results Tree, Summary Report e Aggregate Report.

## Como rodar

Suba a API (porta 3000):

```bash
git clone https://github.com/tiagotaq/api-de-pets.git
cd api-de-pets
npm install
npm start
```

Abra o `apiDePets.jmx` no JMeter 5.6.3 para ver o plano. Para rodar a carga, de dentro desta pasta:

```bash
jmeter -n -t apiDePets.jmx -l executions/logs/logs.jtl -e -o executions/dashboards
```

Apague a pasta `executions` antes de rodar de novo. O relatório sai em `executions/dashboards/index.html`.

## Última execução

26/11/2026, local: 2.501 requisições, nenhum erro.

| Requisição | Amostras | Média (ms) |
|---|---|---|
| POST /login | 1 | 29 |
| POST /pets - modelo fechado | 250 | 0,9 |
| GET /pets - modelo fechado | 250 | 0,7 |
| POST /pets - modelo aberto | 1000 | 1,7 |
| GET /pets - modelo aberto | 1000 | 0,9 |
