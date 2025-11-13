# GitHub Copilot Instructions

Este arquivo orienta o GitHub Copilot (incluindo Chat e Code) ao trabalhar neste repositório.

## Visão Geral do Projeto

Este é um **monorepo pnpm** contendo um aplicativo de inspeção de webhooks com dois pacotes:

- **api**: API REST baseada em Fastify com banco PostgreSQL (Drizzle ORM)
- **web**: SPA em React usando Vite e TanStack Router

A aplicação captura e inspeciona requisições de webhook recebidas, armazenando detalhes como: headers, body, query params, endereço IP e outros metadados.

## Orientações Gerais para o Copilot

- Considere sempre que o projeto é um **monorepo pnpm**.
- Prefira comandos `pnpm` em vez de `npm` ou `yarn`.
- Ao sugerir mudanças, leve em conta:
  - Padrões de rotas existentes (tanto no `api` quanto no `web`).
  - Uso de TypeScript fortemente tipado (tipos, generics, etc.).
  - Uso de Biome para formatação e lint.

## Setup de Desenvolvimento

### Pré-requisitos

- `pnpm` 10.15.0 (gerenciado via `packageManager` no `package.json` raiz)
- Docker (para banco de dados PostgreSQL)

### Passos Iniciais

Ao sugerir scripts de setup ou onboarding, siga esta ordem:

```bash
# Instalar dependências (no diretório raiz)
pnpm install

# Iniciar o banco PostgreSQL (a partir de api/)
cd api
docker compose up -d

# Gerar e rodar migrações
pnpm db:generate
pnpm db:migrate

# Opcional: popular com dados de exemplo
pnpm db:seed
```

## Comandos Comuns

Ao sugerir scripts de execução, testes ou formatação, use:

### API (no diretório `api/`)

- `pnpm dev` - Inicia o servidor de desenvolvimento com hot reload (porta 3333)
- `pnpm format` - Formata o código com Biome
- `pnpm db:generate` - Gera migrações do Drizzle a partir de mudanças no schema
- `pnpm db:migrate` - Aplica migrações pendentes no banco
- `pnpm db:studio` - Abre o Drizzle Studio (GUI do banco)
- `pnpm db:seed` - Executa seed de dados

### Web (no diretório `web/`)

- `pnpm dev` - Inicia o servidor de desenvolvimento do Vite
- `pnpm build` - Faz type-check e build de produção
- `pnpm preview` - Faz preview do build de produção localmente
- `pnpm format` - Formata o código com Biome

### Raiz do Monorepo

- Em geral, **não** sugerir rodar comandos de app a partir da raiz.
- Preferir executar comandos nos workspaces específicos: `api/` ou `web/`.

## Arquitetura

### API (`api/`)

**Stack**: Fastify + Drizzle ORM + PostgreSQL + Zod + TypeScript

**Arquivos importantes** (ao navegar/sugerir código):

- `src/server.ts`  
  - Setup do servidor Fastify
  - Configuração de Swagger, CORS e type providers
- `src/routes/`  
  - Handlers de rotas seguindo o padrão de plugin do Fastify  
  - Cada arquivo de rota exporta um `FastifyPluginAsyncZod`
- `src/db/schema/`  
  - Schemas de tabelas do Drizzle (atualmente `webhooks`)
- `src/db/migrations/`  
  - SQL de migrações auto-geradas
- `src/db/index.ts`  
  - Conexão com banco e instância do Drizzle
- `src/env.ts`  
  - Validação de variáveis de ambiente

**Padrão de rotas**:

- Cada arquivo de rota é um plugin Fastify (`FastifyPluginAsyncZod`).
- Utilizar Zod para:
  - Validação de request/response
  - Geração automática de documentação OpenAPI/Swagger
- Priorizar handlers type-safe para `request` e `reply`.

**Banco de dados**:

- Drizzle ORM com dialeto PostgreSQL.
- Schema principal em `src/db/schema/webhooks.ts`.
- Convenção de nomes de tabela: **snake_case** (via `drizzle.config.ts`).
- Primary keys utilizam UUIDv7 para IDs ordenados temporalmente.
- A tabela `webhooks` armazena:
  - `method`, `pathname`, `ip`, `statusCode`
  - `headers` (JSONB)
  - `body`
  - `queryParams` (JSONB)
  - e outros metadados relevantes de requisições webhook.

**Documentação da API**:

- Endpoint Swagger/Scalar UI em: `http://localhost:3333/docs`

Ao sugerir novas rotas, seguir rigorosamente o padrão existente:
- Plugin Fastify isolado por arquivo.
- Schemas Zod completos (params, query, body, response).
- Tipagem consistente com Drizzle.

### Web (`web/`)

**Stack**: React 19 + Vite + TanStack Router + Tailwind CSS v4 + TypeScript

**Arquivos importantes**:

- `src/main.tsx`  
  - Ponto de entrada da aplicação e setup do router.
- `src/routes/`  
  - Rotas baseadas em arquivos (file-based routing) com TanStack Router.
- `src/routeTree.gen.ts`  
  - Árvore de rotas auto-gerada (**não editar manualmente**).
- `src/components/`  
  - Componentes React da aplicação.
- `src/components/ui/`  
  - Componentes de UI reutilizáveis.

**Router**:

- Usa TanStack Router com file-based routing.
- A árvore de rotas é gerada automaticamente pelo plugin Vite: `@tanstack/router-plugin`.
- Ao criar novas telas, preferir criar novos arquivos em `src/routes/` em vez de registrar rotas manualmente.

**Estilização**:

- Tailwind CSS v4 com classes utilitárias.
- Uso de `tailwind-merge` e `tailwind-variants` para variantes de componentes.
- Manter consistência de nomenclatura de classes e variantes quando sugerir novos componentes de UI.

**Bibliotecas de UI**:

- **Radix UI** para componentes com acessibilidade (ex.: checkbox).
- **Lucide React** para ícones.
- **react-resizable-panels** para layouts com painéis redimensionáveis.
- **Shiki** para syntax highlighting.

Ao sugerir novos componentes:
- Preferir composição com os componentes existentes em `src/components/ui/`.
- Manter padrão de acessibilidade (usar Radix quando aplicável).

## Estilo de Código

- Usar **Biome** para formatação e lint (não usar Prettier/ESLint).
- A configuração está nos arquivos `biome.json` dos pacotes.
- Ao sugerir alterações, seguir:
  - Convenções de import (ordem, tipos separados se configurado).
  - Padrão de aspas, vírgulas, quebras de linha, etc. de acordo com Biome.

## Workflow de Banco de Dados

Ao sugerir mudanças de schema, seguir **sempre** este fluxo:

1. Alterar arquivos de schema em `src/db/schema/`.
2. Rodar `pnpm db:generate` para gerar as migrações SQL.
3. Rodar `pnpm db:migrate` para aplicar as migrações.
4. Revisar o SQL gerado em `src/db/migrations/` antes de commitar.

Evitar:
- Editar migrações já aplicadas.
- Criar tabelas ou colunas diretamente via SQL sem refletir no schema do Drizzle.

## Notas Importantes

- Sempre usar `pnpm` (e não `npm`) porque o monorepo usa **pnpm workspaces**.
- O servidor da API roda na porta `3333` com CORS habilitado para desenvolvimento local.
- O banco de dados roda em Docker na porta `5432` (`user: docker`, `password: docker`).
- Arquivos de rota (tanto na API quanto no Web) seguem padrões específicos:
  - **API**: plugins Fastify com Zod e documentação integrada.
  - **Web**: file-based routing com TanStack Router.
- A árvore de rotas (`src/routeTree.gen.ts`) é gerada automaticamente e **não deve ser editada manualmente**.
- Ambos os pacotes compartilham a mesma versão de TypeScript (~5.9.3).

Ao sugerir novas funcionalidades:
- Respeitar a arquitetura já existente.
- Reusar padrões de tipagem, validação e organização de arquivos observados no código atual.
- Manter separação clara entre responsabilidades da `api` e do `web`.