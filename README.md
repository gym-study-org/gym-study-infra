<h1 align="center">
  Gym Study · Infra
</h1>

<p align="center">
  <img src="docs/arch.gif" alt="Arquitetura do Gym Study: navegador, front Next.js, API Express, PostgreSQL, Redis, MinIO e serviços externos" />
</p>

<p align="center">
  <a href="https://skillicons.dev">
    <img src="https://skillicons.dev/icons?i=docker,postgres,redis,nodejs,nextjs" alt="Stacks" />
  </a>
</p>

## Qual a finalidade do projeto?

Sobe a stack inteira do **Gym Study**, a plataforma de estudos gamificada, com um único `docker compose up`: web app, API, banco, cache e filas, e armazenamento de arquivos.

As imagens do front e do back são construídas a partir dos repositórios vizinhos ([gym-study-front](https://github.com/gym-study-org/gym-study-front) e [gym-study-back](https://github.com/gym-study-org/gym-study-back)), então os três precisam estar clonados lado a lado.

## O que foi construído

### Serviços

| Serviço | Imagem | Porta | Função |
|---|---|---|---|
| `frontend` | build de `../gym-study-front` | 3000 | Web app Next.js |
| `backend` | build de `../gym-study-back` | 5000 | API Express + Socket.io, aplica as migrations ao subir |
| `postgres` | `postgres:16-alpine` | 5432 | Banco de dados |
| `redis` | `redis:7-alpine` | 6379 | Cache, filas BullMQ e pub/sub do Socket.io |
| `minio` | `cgr.dev/chainguard/minio` | 9000 / 9001 | Avatares (API S3) e console web |

Todos os serviços têm healthcheck, e o `backend` só sobe depois que banco, Redis e MinIO estão saudáveis.

### Arquivos

| Arquivo | Uso |
|---|---|
| `docker-compose.yml` | Stack em modo produção |
| `docker-compose.dev.yml` | Override com hot reload (monta o `src` dos repositórios) |
| `.env.example` | Variáveis de ambiente (JWT, banco, SMTP, OAuth, URLs) |
| `docker/postgres/init.sql` | Script executado na criação do banco |

### Volumes

`postgres_data`, `redis_data`, `minio_data` e `backend_logs`, todos na rede `gym-study-network`.

## Tecnologias utilizadas

- **Docker Compose:** orquestração local dos cinco serviços;
- **PostgreSQL 16:** banco relacional;
- **Redis 7:** cache e filas;
- **MinIO:** armazenamento compatível com S3;
- **Node.js 20 e Next.js 14:** imagens do back e do front.

## Estrutura do repositório

```text
gym-study-infra/
├── docker/postgres/init.sql
├── docs/arch.gif              # Diagrama da arquitetura
├── docker-compose.yml
├── docker-compose.dev.yml
├── .env.example
└── README.md
```

## Fluxo de funcionamento

1. `docker compose up` sobe PostgreSQL, Redis e MinIO e espera o healthcheck de cada um.
2. O `backend` aplica as migrations no banco, cria o bucket no MinIO e começa a atender em `:5000`.
3. O `frontend` sobe em `:3000`, já compilado com a URL da API.
4. No navegador, o front chama a API por REST e WebSocket.
5. A API grava no PostgreSQL, usa o Redis para cache e filas, e guarda os avatares no MinIO.

## Como rodar

```bash
git clone https://github.com/gym-study-org/gym-study-front
git clone https://github.com/gym-study-org/gym-study-back
git clone https://github.com/gym-study-org/gym-study-infra
cd gym-study-infra

cp .env.example .env            # troque o JWT_SECRET (mínimo 32 caracteres)
docker compose up -d --build
node ../gym-study-back/scripts/seed-demo.mjs   # dados de exemplo (opcional)
```

| Endereço | O quê |
|---|---|
| http://localhost:3000 | Web app (login `ana@gymstudy.dev` / `Demo1234` depois do seed) |
| http://localhost:5000/docs | Swagger UI da API |
| http://localhost:9001 | Console do MinIO |

Modo desenvolvimento, com hot reload:

```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
```

Para apagar tudo, inclusive os dados: `docker compose down -v`.

## Como validar a entrega

Em uma validação end-to-end, a stack deve subir do zero, com o banco vazio, e o front deve conseguir cadastrar um usuário e registrar uma sessão de estudo.

Pontos principais de validação:

- `docker compose ps` com os cinco serviços `healthy`;
- logs do `backend` mostrando as 29 migrations aplicadas e o bucket `gym-study` criado;
- `GET http://localhost:5000/health` respondendo `200`;
- front abrindo em `:3000` e chamando a API em `:5000`;
- cadastro, login e registro de sessão funcionando pelo navegador.

## Projeto Gym Study

| Repositório | Camada |
|---|---|
| [gym-study-front](https://github.com/gym-study-org/gym-study-front) | Web app (Next.js) |
| [gym-study-back](https://github.com/gym-study-org/gym-study-back) | API (Express + PostgreSQL + Redis) |
| **gym-study-infra** | Stack completa com Docker Compose |

## Autor

**William Alves Coelho** · [@willtechdev](https://github.com/willtechdev)
