# Webhook Inspector

Aplicação composta por uma API Fastify e uma SPA React para captura e inspeção de requisições de webhook. O repositório é um monorepo gerenciado por pnpm com dois workspaces: `api/` e `web/`.

## Estrutura do Projeto

- `api/`: Fastify + Drizzle ORM + PostgreSQL.
- `web/`: React 19 + Vite + TanStack Router + Tailwind CSS v4.

## Requisitos

- [pnpm](https://pnpm.io/) 10.15.0 (instalado automaticamente via `corepack enable`).
- Docker com suporte a Compose (utilizado para o banco PostgreSQL).

## Instalação

Instale as dependências somente uma vez na raiz:

```bash
pnpm install
```

## Banco de Dados

```bash
cd api
# Sobe PostgreSQL local
docker compose up -d
# Gera e aplica migrações
pnpm db:generate
pnpm db:migrate
# (Opcional) Popula com dados de exemplo
pnpm db:seed
```

## Uso

### API

```bash
cd api
pnpm dev
```

- Servidor em `http://localhost:3333`.
- Documentação Swagger/Scalar em `http://localhost:3333/docs`.

### Web

```bash
cd web
pnpm dev
```

- SPA acessível em `http://localhost:5173` (ou porta indicada pelo Vite).

## Tecnologias

- pnpm workspaces
- Fastify, Zod, Drizzle ORM, PostgreSQL
- React 19, Vite, TanStack Router, Tailwind CSS v4
- Docker, Biome
