# Testes de performance da API de Pets com JMeter

Trabalho da disciplina de Automação de Testes de Performance (PGATS). Plano de teste do JMeter para a [API de Pets](https://github.com/tiagotaq/api-de-pets) com um thread group de modelo fechado e um de modelo aberto. Os dois fazem `POST /pets` e `GET /pets` depois do login.

## Estrutura

```text
apiDePets.jmx        plano de teste
fixtures/pets.csv    nomes e espécies usados no cadastro dos pets
```

## O que o plano faz

- **HTTP Request Defaults** com `http://localhost:3000` e **HTTP Header Manager** com `Content-Type: application/json`, valendo para todas as requisições.
- **setUp Thread Group - login**: 1 usuário faz o `POST /login` com o usuário `tiago`. O JSON Extractor guarda o `$.token` na variável `tokenExtraido` e o JSR223 PostProcessor passa o token para uma propriedade (`props.put("tokenProp", vars.get("tokenExtraido"))`), porque variável não passa de um thread group para outro.
- **Thread Group - modelo fechado**: 50 usuários, ramp-up de 10 s e 5 iterações.
- **Open Model Thread Group - modelo aberto**: chegadas aleatórias subindo de 0 a 20 req/s em 10 s, 40 s em 20 req/s e 10 s descendo até 0:
  `rate(0) random_arrivals(10 sec) rate(20/sec) random_arrivals(40 sec) random_arrivals(10 sec) rate(0)`
- Nos dois grupos:
  - CSV Data Set Config lendo `./fixtures/pets.csv` com caminho relativo, para rodar em qualquer máquina que clonar o repositório.
  - HTTP Header Manager com `Authorization: Bearer ${__property(tokenProp)}`.
  - `POST /pets` com o nome `${Nome}_${__time()}_${__counter(false)}`, porque a API não aceita dois pets com o mesmo nome para o mesmo usuário.
  - `GET /pets`.
- Response Assertion no código de status: 200 no login e no `GET /pets`, 201 no `POST /pets`.
- Listeners: View Results Tree, Summary Report e Aggregate Report.

## Como rodar

1. Suba a API de Pets, que roda na porta 3000:

```bash
git clone https://github.com/tiagotaq/api-de-pets.git
cd api-de-pets
npm install
npm start
```

2. Abra o `apiDePets.jmx` no JMeter (5.6.3) para ver ou depurar o plano.

3. Para a carga, rode em linha de comando de dentro da pasta deste projeto:

```bash
jmeter -n -t apiDePets.jmx -l executions/logs/logs.jtl -e -o executions/dashboards
```

A pasta `executions` precisa estar vazia ou não existir antes de cada execução. O relatório fica em `executions/dashboards/index.html`.

## Execução de referência

Local, em 26/11/2026: 2.501 requisições em 1 minuto, sem erros.

| Requisição | Amostras | Média (ms) | Erros |
|---|---|---|---|
| POST /login | 1 | 29 | 0 |
| POST /pets - modelo fechado | 250 | 0,9 | 0 |
| GET /pets - modelo fechado | 250 | 0,7 | 0 |
| POST /pets - modelo aberto | 1000 | 1,7 | 0 |
| GET /pets - modelo aberto | 1000 | 0,9 | 0 |
