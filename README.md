# E-commerce — Projeto 3

Este é o e-commerce do Projeto 2 já resolvido, com InversifyJS e os modos `dev` e `test`. Ele é o ponto de partida deste trabalho. Se você fez o Projeto 2, pode usar o seu código.

## Como rodar
É preciso ter o Docker Desktop instalado e aberto.

```bash
cp .env.example .env
docker compose up --build
```

O `docker compose up --build` sobe dois containers:

* `banco`: o PostgreSQL. Ele não publica porta: só a `api` acessa, pela rede interna do Docker.
* `api`: o projeto. Ela só sobe depois que o banco está pronto (`healthcheck` + `depends_on`). Ao subir, o `npm start` cria as tabelas (`prisma db push`), cadastra os produtos e liga o servidor.

A API fica em `http://localhost:3000`.

### Testar

Criar um pedido:

```bash
curl -X POST http://localhost:3000/orders -H "Content-Type: application/json" -d '{"customer":"cliente@ethereal.email","items":[{"productId":1,"quantity":1}],"paymentMethod":"pix"}'
```

Buscar o pedido pelo id (troque o `1` pelo `orderId` que voltou):

```bash
curl -i http://localhost:3000/orders/1
```
Um id que não existe vai devolver 404;

```bash
curl -i http://localhost:3000/orders/999
```

### Os dois modos

O modo é escolhido pela variável `APP_ENV`, no arquivo `.env`:

* `APP_ENV=dev`: os pedidos vão para o PostgreSQL e o e-mail é enviado pelo Ethereal.
* `APP_ENV=test`: os pedidos ficam na memória e o e-mail só aparece no terminal. Funciona sem internet.

Para trocar de modo, mude o `APP_ENV` no `.env` e suba de novo:

```bash
docker compose down
docker compose up --build
```

### Desligar

```bash
docker compose down

```

Os pedidos ficam guardados no volume `dados-banco`: depois de um `docker compose down` e um novo `up`, eles continuam lá. Para apagar os dados também, use `docker compose down -v`.

## Estrutura

```
src/
├── container/     # o único lugar com classes concretas
├── domain/        # produtos, pedido e erros de negócio
├── repositories/  # interfaces e implementações (Prisma e memória)
├── providers/     # e-mail (Ethereal e console)
├── payments/      # formas de pagamento
├── services/      # regras de negócio
├── controllers/   # só HTTP
└── app.ts         # rotas
```

---

## Trabalho

Hoje o projeto usa SQLite e roda direto na máquina. Coloque a aplicação e o banco em containers, troque o SQLite pelo PostgreSQL e crie uma rota nova respeitando as camadas.

### Já pronto

* `Dockerfile` e `.dockerignore` da API.
* O script `npm start`, que cria as tabelas, cadastra os produtos e sobe o servidor. É o comando que o `Dockerfile` executa.
* O `.env.example`, com o `DATABASE_URL` apontando para o serviço `banco`.

### 1. Troque o SQLite pelo PostgreSQL

* No `prisma/schema.prisma`, mude o `provider` para `postgresql` e a `url` para `env("DATABASE_URL")`.
* Apague a pasta `prisma/migrations`: ela foi gerada para o SQLite. As tabelas são criadas pelo `npm start`, com `prisma db push`.

Só a infraestrutura muda. Nenhum arquivo de `services/` ou `controllers/` pode ser alterado por causa dessa troca.

### 2. Rode tudo com Docker Compose

* Crie um `compose.yaml` com dois serviços: `banco` (PostgreSQL) e `api`, que usa o `Dockerfile` pronto.
* O `banco` **não precisa publicar porta**.
* A `api` só pode subir depois que o banco estiver pronto (`healthcheck` + `depends_on`).
* Os dados ficam num volume: `docker compose down` e `up` de novo não apagam os pedidos.

### 3. Crie a rota `GET /orders/:id`

Ela devolve o pedido com o `id` informado, ou **404** se ele não existir.

A rota deve passar pelas três camadas:

```
OrderController  ->  OrderService  ->  IOrderRepository
```

* O controller só trata HTTP: lê o `id` e devolve 200 ou 404.
* O service tem a regra: pedido inexistente é erro.
* O repositório busca o pedido. Implemente o `findById` nos dois repositórios, o do Prisma e o da memória.

A rota tem de funcionar nos dois modos, `dev` e `test`.

### Critérios de aceite

1. Clonar, `cp .env.example .env` e `docker compose up --build` sobem tudo sem erro.
2. `POST /orders` cria o pedido e `GET /orders/:id` o devolve. Um `id` que não existe devolve 404.
3. Nenhum arquivo de `controllers/` acessa um repositório ou o Prisma.
4. A porta do PostgreSQL não está publicada no `compose.yaml`.
5. Uma seção **"Como rodar"** no seu README explica os comandos.

### Entrega

Pelo git. Informe na caixa de texto do AVA o link do repositório. Ele não pode estar privado: teste antes de entregar.
